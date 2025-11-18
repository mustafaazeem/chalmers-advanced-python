# Installing graphviz on Chalmers computer

## Windows

1. Download Graphviz ZIP archive from <https://graphviz.org/download/>
2. Extract to a folder somewhere, e.g. `C:\Users\cajohn\Downloads\Graphviz-14.0.4-win64`
3. Add the `bin` subfolder to your system PATH (for running from shell)
    - cmd: `PATH %PATH%;C:\Users\cajohn\Downloads\Graphviz-14.0.4-win64\bin`
    - PowerShell: `$env:Path += ';C:\Users\cajohn\Downloads\Graphviz-14.0.4-win64\bin'`
4. Add this to your VS Code (for integrated terminal):
    - Open `settings.json`
    - Add this:

      ```json
      "terminal.integrated.env.windows": {
        "PATH": "C:\\Users\\cajohn\\Downloads\\Graphviz-14.0.4-win64\\bin;${env:PATH}",
      },
      ```
