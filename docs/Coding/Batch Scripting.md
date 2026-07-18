# Useful Batch Scripting Hacks

## What Is a Batch Script?
A way to orchestrate a series of processes into a .bat file, where you can just 'double click' and execute the process, without opening VSCode. Batch scripts are:

- Available on every Windows machine
- Easy to distribute
- Fast to write
- Ideal for launching applications and orchestrating a sequence of scripts. 


## Always Start with `@echo off`

```batch
@echo off
```

This prevents every command from being printed to the console before execution, resulting in much cleaner output.

Example:

```batch
@echo off

echo Starting process...
# do something here

echo Complete.
```

Output:

```text
Starting process...
Complete.
```

---

## Use `REM` to Comment

```batch
@echo off

REM Set working directory
cd C:\Projects

echo Running analysis...
```
Or you can use `::` to comment:

```batch
:: This is a comment. 
```

---

## Using Variables

Instead of repeating file paths:

```batch
copy C:\Projects\Data\input.csv C:\Backup\
```

Use variables:

```batch
set PROJECT_DIR=C:\Projects\Data
set BACKUP_DIR=C:\Backup

copy "%PROJECT_DIR%\input.csv" "%BACKUP_DIR%"
```

---

## Always Quote File Paths

```batch
copy "C:\My Files\data.csv" "C:\Backup"
```


---

## Pause Before the Window Closes - SUPER HELPFUL!

If we error, the script won't close before you get a chance to see the error. 

```batch
pause
```
---

## Check for Errors Using `%ERRORLEVEL%`

Many commands return an exit code.

```batch
robocopy Source Destination
```

Check whether it succeeded:

```batch
if %ERRORLEVEL% neq 0 (
    echo Operation failed.
)
```

Or:

```batch
if errorlevel 1 (
    echo An error occurred.
)
```

---

## Create Menus

```batch
@echo off

echo =======================
echo     MAIN MENU
echo =======================
echo 1. Run Analysis
echo 2. Backup Files
echo 3. Exit

set /p choice=Enter choice:

if "%choice%"=="1" goto Analysis
if "%choice%"=="2" goto Backup
if "%choice%"=="3" exit

:Analysis
echo Running analysis...
goto End

:Backup
echo Running backup...
goto End

:End
pause
```

---

## Loop Through Files

Process every file in a directory:

```batch
for %%f in (*.csv) do (
    echo Processing %%f
)
```

---

## Timestamp Your Files

```batch
set DATESTAMP=%DATE:~-4%%DATE:~3,2%%DATE:~0,2%

echo Report > Report_%DATESTAMP%.txt
```
---

## Redirect Console Output to a Text File

One of the most useful batch scripting techniques is logging script output for debugging. 

### Redirect Standard Output

```batch
my_script.bat > log.txt
```

This sends all console output to `log.txt`.

Example:

```batch
echo Starting process... > log.txt
dir >> log.txt
echo*Complete. >> log.txt
```

The first `>` creates/overwrites the file.
The `>>` operator appends additional output.

---

### Log to a Temporary File, Then Convert to `.txt`

This is useful when you want to continuously log script output, then convert it to a `.txt` file at the end.

```batch
@echo off

set TMP_LOG=run.tmp
set FINAL_LOG=run.txt

echo Script started at %DATE% %TIME% > "%TMP_LOG%"

echo Processing file 1...
echo Processing file 1... >> "%TMP_LOG%"

timeout /t 5 >nul

echo Processing file 2...
echo Processing file 2... >> "%TMP_LOG%"

timeout /t 5 >nul

echo Script completed successfully. >> "%TMP_LOG%"
```
So when we run it, the contents of `run.tmp` will get populated with the script outputs continuously. 

Lastly, we can add a line to rename the temp file to `.txt` when finished.

```batch
ren "%TMP_LOG%" "%FINAL_LOG%"
```
---


## Call `exe`s

Probably wouldn't recommend actually doing that. But instead we can call FME.exe 😉

```batch
start fme.exe
```

Or if you have an FME workbench configured to take in input arguments, you can pipe in arguments directly into the workbench. This is equivalent to opening the workspace in FME Form and clicking Run.

```batch

@echo off

set INPUT=C:\Data\Input.gpkg
set OUTPUT=C:\Results

"C:\Program Files\FME\fme.exe" ^
    "C:\Workspaces\ConvertData.fmw" ^
    --INPUT_FILE "%INPUT%" ^
    --OUTPUT_FOLDER "%OUTPUT%"


```

---


## For loops and FME with Batch Scripting
Let's say I want to call a workspace repeatedly on multiple input files. Suppose we have a Workbench we want to iterateively run. 

Yes we can build a parent workspce, but that requires opening FME and I'm lazy. So we can just make a `.bat` file that's double clickable. 

```batch
@echo off

REM loop through everything in C:\Data that ends in DWG. 
for %%F in (C:\Data\*.dwg) do (

    echo Processing %%~nxF
    
    REM Same as above, where I'm itratively calling the FME Workbench on here. 
    "C:\Program Files\FME\fme.exe" ^
        "C:\Workspaces\ConvertData.fmw" ^
        --INPUT_FILE "%%F" ^
        --OUTPUT_FOLDER "C:\Output"

)
```
