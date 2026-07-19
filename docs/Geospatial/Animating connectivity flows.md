#  Animating connectivity flows

Recently, I gave a [talk](https://matthewgarethlam.github.io/thespacebetween/Talks/A%20flapping%20good%20model/) at a conference about habitat suitability and connectivity modelling. For connectivity, we used Circuitscape (separate post about this [here](https://matthewgarethlam.github.io/thespacebetween/Geospatial/Connectivity%20modelling/)). I didn't want to just show some static rasters on screen, so decided to experiment with `matplotlib` animations to give my circuitscape current rasters a little bit more life.

The gif below shows the connectivity corridors for a species of bat. The raster that comes on as we progress is the circuitscape cumulative current output. 

![alt text](assets/Barbastella%20Movement_animation.gif)

Here's how! 


## Libraries
```Python
import numpy as np
import rasterio
import matplotlib.pyplot as plt
from matplotlib import animation
from scipy import ndimage
from scipy import stats
from rasterio.mask import mask
import os
import geopandas as gpd
import math
```


## Load in the raster (Circuitscape output)
```Python
raster_path = "barbastelle_ra.tif"

# Read the raster data
def load_raster(path):
    with rasterio.open(path) as src:
        data = src.read(1)
        profile = src.profile
    return data, profile

raster, raster_profile = load_raster(raster_path)


dummy_aoi = gpd.read_file("dummy_aoi2_for_cieem_Po.shp")

#clip the raster to the aoi
with rasterio.open(raster_path) as src:
    aoi = dummy_aoi.to_crs(src.crs) if (src.crs is not None and dummy_aoi.crs != src.crs) else dummy_aoi
    clipped, clipped_transform = mask(src, aoi.geometry, crop=True)
    raster = clipped[0]  # convert (1, rows, cols) -> (rows, cols)

    raster_profile = src.profile.copy()
    raster_profile.update({
        "height": raster.shape[0],
        "width": raster.shape[1],
        "transform": clipped_transform
    })

# pplot
def plot_raster(raster):
    plt.figure(figsize=(8, 6))
    plt.imshow(raster, cmap='viridis')
    plt.colorbar(label='Movement Intensity')
    plt.title('Bat Migration Movement Raster')
    plt.axis('off')
    plt.show()

#remove 0s
raster[raster <= 0] = np.nan
# Remove max value (ignoring nan)
raster[raster == np.nanmax(raster)] = np.nan
plot_raster(raster)

```
![alt text](assets\01_raster_raw.png)


## Enhancing the contrast of the connectivity flows

The flows are quite hard to see - so we might want to do some exaggeration so that we can better separate the values in the histogram. 

```Python
#histogram of raster values
plt.figure(figsize=(8, 6))
plt.hist(raster[~np.isnan(raster)].flatten(), bins=50, color='blue', alpha=0.7)
plt.title('Histogram of Raster Values')
plt.xlabel('Raster Value')
plt.ylabel('Frequency')
plt.show()
```
![alt text](assets\01_orig_hist.png)


Here, we add a negative exponential transform to enhance the contrast
```Python
mean_value = np.nanmean(raster)

raster_transformed = np.exp(-raster / mean_value)

#Flipping it so that higher values are higher current. 
flipped_raster = 1-raster_transformed

plot_raster(flipped_raster)
```
![alt text](assets\01_raster_enhanced.png)


We add a threshold to show only where movement intensity above 0.6 is shown (for better visualisation) and our histogram now looks like this

```Python
#raster where only movement intensity above 0.6 is shown 
raster_thresholded = np.where(flipped_raster > 0.6, flipped_raster, np.nan)


#histogram of transformed raster values
plt.figure(figsize=(8, 6))
plt.hist(raster_thresholded[~np.isnan(raster_thresholded)].flatten(), bins=50, color='blue', alpha=0.7)
plt.title('Histogram of Transformed Raster Values')
plt.show()
```
![alt text](assets\01_thresholded_hist.png)

## Generating Trajectories

What I do here is: 

1. Generate random "particles" to spawn in the raster 
2. Allow the particles to move in the raster, following the raster gradient (flow field) with some level of random movement. 
3. Track the particles as they slow and store them as trajectories. 


```Python
def generate_particle_flows(raster, num_particles=100, num_steps=100, step_size=1, random_prob=0.15, min_dist=10):
    """
    Generate particle trajectories that follow the raster gradient (flow field), with some random movement.
    Ensures starting points are not too close together (min_dist in pixels).
    """
    # Raster dimensions (rows = y, cols = x)
    rows, cols = raster.shape

    # Candidate starting pixels: only use higher-intensity cells
    # (acts as a mask so particles start in "important" areas)
    valid_indices = np.argwhere(raster > 0.5)

    # Randomize candidate order so selected seeds are spatially varied
    np.random.shuffle(valid_indices)

    # Greedy seed selection with minimum spacing between particles
    selected = []
    for idx in valid_indices:
        if len(selected) == 0:
            # First point is always accepted
            selected.append(idx)
        else:
            # Compute distances from this candidate to all selected points
            dists = np.linalg.norm(np.array(selected) - idx, axis=1)

            # Keep candidate only if it is far enough from all selected points
            if np.all(dists >= min_dist):
                # Accept this candidate because it is at least `min_dist` pixels
                # away from all previously selected starting points.
                # This keeps seed points spatially distributed (less clustering).
                selected.append(idx)

            # Stop once enough particles have been seeded
            if len(selected) >= num_particles:
                break

    if len(selected) < num_particles:
        print(f'Warning: Only {len(selected)} starting points found with min_dist={min_dist}')

    # Final seed array (row, col), limited to requested particle count
    start_indices = np.array(selected)[:num_particles]

    # Trajectory tensor: [particle_id, time_step, (x, y)]
    trajectories = np.zeros((len(start_indices), num_steps, 2))

    # Initialize t=0 positions (convert row/col -> x/y = col/row)
    trajectories[:, 0, :] = start_indices[:, [1, 0]]

    # Precompute raster gradient (flow direction field)
    # grad_x: change along x (columns), grad_y: change along y (rows)
    grad_y, grad_x = np.gradient(raster)

    # Simulate motion over time
    for t in range(1, num_steps):
        for i in range(len(start_indices)):
            # Previous particle location
            current_pos = trajectories[i, t - 1, :]
            x, y = current_pos[0], current_pos[1]

            # Convert to nearest pixel index for raster lookup
            xi, yi = int(round(x)), int(round(y))

            # Move only if current location is inside raster and not NaN
            if 0 <= yi < rows and 0 <= xi < cols and not np.isnan(raster[yi, xi]):
                # Local gradient vector at current pixel
                gx = grad_x[yi, xi]
                gy = grad_y[yi, xi]

                # Normalise gradient so step size controls movement magnitude
                norm = np.hypot(gx, gy)
                if norm > 0:
                    dx = gx / norm
                    dy = gy / norm
                else:
                    # Flat area: no preferred direction
                    dx, dy = 0, 0

                # Optional random perturbation (adds wandering behavior)
                if np.random.rand() < random_prob:
                    dx += np.random.uniform(-1, 1)
                    dy += np.random.uniform(-1, 1)

                # Advance position by scaled direction vector
                new_x = x + step_size * dx
                new_y = y + step_size * dy

                # Clamp to raster extent to avoid out-of-bounds indices
                new_x = np.clip(new_x, 0, cols - 1)
                new_y = np.clip(new_y, 0, rows - 1)

                # Accept move only if destination is valid (not NaN); otherwise stay put
                if not np.isnan(raster[int(round(new_y)), int(round(new_x))]):
                    trajectories[i, t, :] = [new_x, new_y]
                else:
                    trajectories[i, t, :] = [x, y]
            else:
                # If current state is invalid, keep the same position
                trajectories[i, t, :] = [x, y]

    return trajectories

# Generate trajectories that follow the raster flow field, with some random movement and well-separated starting points
trajectories = generate_particle_flows(flipped_raster, num_particles=600, num_steps=1000, step_size=10, random_prob=0.9, min_dist=100)
```


So what does this give us? 

```Python
plt.figure(figsize=(8, 6))
plt.style.use('dark_background')  # Set black background
plt.imshow(flipped_raster, cmap='magma', alpha = 0.5)
for i in range(trajectories.shape[0]):
    #randomly set alpha and linewidth for each line for visual variety
    alpha = np.random.uniform(0.1, 0.9, size=1)[0]
    linewidth = np.random.uniform(0.15,0.5, size=1)[0]
    print(f'Plotting trajectory {i} with alpha={alpha:.2f} and linewidth={linewidth:.2f}')
    plt.plot(trajectories[i, :, 0], trajectories[i, :, 1], alpha=alpha, linewidth=linewidth, color='white')
plt.show()
```
![alt text](assets\01_Barbastella_movement_initial_plot.png)



## Animating the trajectories

The key idea here is that now we have all our trajectories calcualted, we want to sequentially visualise them in some type of order. 

`matplotlib` has some great functions to animate graphics. 

```Python
def animate_flows(raster, trajectories, title, output_dir=r'Outputs', fps=30, interval=20, tail_length=100, linewidth=0.2, fig_width=16, fig_height=12, fadein_seconds=3):

    # Normalise title so output naming is consistent (strip path and keep first token before "_")
    title = title.split('\\')[-1].split('_')[0]

    # Build output GIF path
    output_gif = os.path.join(output_dir, f'{title}_animation.gif')
    print(f'Animating {title} and saving to {output_gif}...')

    # Create plotting canvas and apply dark style
    fig, ax = plt.subplots(figsize=(fig_width, fig_height))
    plt.style.use('dark_background')
    
    # Draw raster in the background but start fully transparent (it will fade in later)
    raster_img = ax.imshow(raster, cmap='magma', alpha=0.0)

    # Number of trajectory lines to animate
    n_lines = trajectories.shape[0]

    # Give each line a slightly different opacity and thickness for visual variation
    alphas = np.random.uniform(0.9, 1, size=n_lines)
    linewidths = np.random.uniform(0.3, linewidth, size=n_lines)

    
    # Colour palette used to map each species/title to a single trajectory colour
    path_col = ["white", "cyan" , "lime", "yellow" , "magenta" ]
    # Assign line colour based on the title/species label
    if title == 'Barbastella Movement':
        color = path_col[0]
    elif title == 'Myotis Movement':
        color = path_col[1]
    elif title == 'Eptesicus Movement':
        color = path_col[2]
    elif title == 'Plecotus Movement':
        color = path_col[3]
    elif title == 'Pipistrellus Movement':
        color = path_col[4]
    else:
        # Safe fallback if title does not match expected labels
        color = "white"

    # Pre-create matplotlib Line2D objects; they will be updated every animation frame
    lines = [ax.plot([], [], color=color, alpha=alphas[i], linewidth=linewidths[i])[0] for i in range(n_lines)]

    # Hide axes so only raster + trajectories are visible
    ax.axis('off')

    # Time dimension per trajectory (number of points along each path)
    n_steps = trajectories.shape[1]
    n_frames = n_steps

    # Convert fade-in duration (seconds) to frame count
    fadein_frames = int(fadein_seconds * fps)

    # Total animation includes trajectory frames plus fade-in staging frames
    total_frames = n_frames + fadein_frames

    def init():
        # Initialise all lines as empty so animation starts from a blank canvas
        for line in lines:
            line.set_data([], [])
        # Keep raster hidden at frame 0
        raster_img.set_alpha(0.0)
        return lines + [raster_img]

    def update(frame):
        # During early frames, keep raster invisible so trajectories appear first
        if frame < fadein_frames:
            raster_img.set_alpha(0.0)
        else:
            # Then gradually ramp raster alpha to 0.6 over fadein_frames
            fade_progress = min(1.0, (frame - fadein_frames) / fadein_frames)
            raster_img.set_alpha(fade_progress * 0.6)

        # Clamp animation index to valid trajectory range once fade stage exceeds n_frames
        t = min(frame, n_frames-1)
        for i, line in enumerate(lines):
            # Draw only a trailing window of points to create a moving "tail" effect
            start = max(0, t - tail_length)
            end = t
            line.set_data(trajectories[i, start:end, 0], trajectories[i, start:end, 1])

            # Add subtle pulsating opacity per line to make motion feel more dynamic
            line.set_alpha(alphas[i] * (0.5 + 0.5 * math.sin(frame / 30 + i)))
        return lines + [raster_img]

    # Build animation object
    anim = animation.FuncAnimation(fig, update, init_func=init, frames=total_frames, blit=True, interval=interval, repeat=True)

    # Save GIF
    anim.save(output_gif, writer='imagemagick', fps=fps)

    plt.close()

animate_flows(raster_transformed, trajectories, 'Barbastella Movement', fps=30, interval=20, tail_length=1000, linewidth=0.8, fig_width=8, fig_height=6, fadein_seconds=5)
```

![alt text](assets/Barbastella%20Movement_animation.gif)