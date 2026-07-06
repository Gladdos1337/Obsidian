## Essential Interactive FTP Client Commands

Once you establish a connection using the `ftp` command, use these inside the interactive prompt:

- **`status`**: Check current settings (shows whether Active or Passive mode is enabled).
- **`passive`**: Toggle Passive mode on/off (crucial if commands like `ls` freeze).
- **`ls -a`**: List all files, including hidden files (like `.ssh` or configuration files).
- **`binary`**: Switch to binary transfer mode (always run this before downloading ZIPs, executables, or images to prevent corruption).
- **`get <filename>`**: Download a single file to your current local working directory.
- **`put <local-file>`**: Upload a file to the remote server (tests for write permissions / web shell placement).
- **`exit` / `bye`**: Terminate the FTP session.