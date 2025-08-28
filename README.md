# Windows Service Example (Key Event Logger)

## 📌 Overview
This project demonstrates how to create a **Windows Service** in C using the Win32 API.  
The service runs in the background and captures key events, writing them to a log file.  

⚠️ **Note:** This project is developed **strictly for educational purposes** as part of a coursework/assignment.  
It must not be used for malicious activity such as spying on users or stealing personal data.  

## 🚀 Features
- Installable Windows Service
- Runs automatically in the background
- Logs key events into a file (`C:\keylog.txt`)
- Graceful service start and stop handling

## 🛠️ How It Works
- The executable can be installed as a service with the argument:
  ```bash
  MyService.exe install
