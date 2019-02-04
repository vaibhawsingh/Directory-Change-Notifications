# Directory Change Notifications
To know when and which files are changed in windows filesystem and getting notification

--------------------------------------------------------------------------------------------------------------------------------

[1. Using ReadDirectoryChangesW function](#1-using-readdirectorychangesw-function)

[2. Using FindFirstChangeNotification function](#2-using-findfirstchangenotification-function)

--------------------------------------------------------------------------------------------------------------------------------

<!-- toc -->
## 1. Using ReadDirectoryChangesW function ##
  - Using ReadDirectoryChangesW function one can obtain all the details related to the file modification. you can call WatchDirectory() function directly from main function.
  
<pre><code>
void WatchDirectory(LPCWSTR path)
{
	char buf[2048];
	DWORD nRet;
	BOOL result = TRUE;
	char filename[MAX_PATH];
	TCHAR lpszDirName[MAX_PATH];
	HANDLE hDir = CreateFile(path, GENERIC_READ | FILE_LIST_DIRECTORY,
		FILE_SHARE_READ | FILE_SHARE_WRITE | FILE_SHARE_DELETE,
		NULL, OPEN_EXISTING, FILE_FLAG_BACKUP_SEMANTICS | FILE_FLAG_OVERLAPPED,
		NULL);

	if (hDir == INVALID_HANDLE_VALUE)
	{
		return; //cannot open folder
	}

	lstrcpy(lpszDirName, path);
	OVERLAPPED PollingOverlap;

	FILE_NOTIFY_INFORMATION* pNotify;
	int offset;
	PollingOverlap.OffsetHigh = 0;
	PollingOverlap.hEvent = CreateEvent(NULL, TRUE, FALSE, NULL);
	while (result)
	{
		result = ReadDirectoryChangesW(
			hDir,// handle to the directory to be watched
			&buf,// pointer to the buffer to receive the read results
			sizeof(buf),// length of lpBuffer
			TRUE,// flag for monitoring directory or directory tree
			FILE_NOTIFY_CHANGE_FILE_NAME |
			FILE_NOTIFY_CHANGE_DIR_NAME |
			FILE_NOTIFY_CHANGE_SIZE,
			//FILE_NOTIFY_CHANGE_LAST_WRITE |
			//FILE_NOTIFY_CHANGE_LAST_ACCESS |
			//FILE_NOTIFY_CHANGE_CREATION,
			&nRet,// number of bytes returned
			&PollingOverlap,// pointer to structure needed for overlapped I/O
			NULL);

		WaitForSingleObject(PollingOverlap.hEvent, INFINITE);
		offset = 0;
		int rename = 0;
		char oldName[260];
		char newName[260];
		do
		{
			pNotify = (FILE_NOTIFY_INFORMATION*)((char*)buf + offset);
			strcpy_s(filename, "");
			int filenamelen = WideCharToMultiByte(CP_ACP, 0, pNotify->FileName, pNotify->FileNameLength / 2, filename, sizeof(filename), NULL, NULL);
			filename[pNotify->FileNameLength / 2] = '\0';
			switch (pNotify->Action)
			{
			case FILE_ACTION_ADDED:
				printf("\nThe file is added to the directory: [%s] \n", filename);
				break;
			case FILE_ACTION_REMOVED:
				printf("\nThe file is removed from the directory: [%s] \n", filename);
				break;
			case FILE_ACTION_MODIFIED:
				printf("\nThe file is modified. This can be a change in the time stamp or attributes: [%s]\n", filename);
				break;
			case FILE_ACTION_RENAMED_OLD_NAME:
				printf("\nThe file was renamed and this is the old name: [%s]\n", filename);
				break;
			case FILE_ACTION_RENAMED_NEW_NAME:
				printf("\nThe file was renamed and this is the new name: [%s]\n", filename);
				break;
			default:
				printf("\nDefault error.\n");
				break;
			}

			offset += pNotify->NextEntryOffset;

		} while (pNotify->NextEntryOffset); //(offset != 0);
	}

	CloseHandle(hDir);

}
<code><pre>

## 2. Using FindFirstChangeNotification function ##

