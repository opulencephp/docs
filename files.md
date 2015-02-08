# File System

## Table of Contents
1. [Introduction](#introduction)
2. [Basic Usage](#basic-usage)
  1. [Reading a File](#reading-a-file)
  2. [Writing to a File](#writing-to-a-file)
  3. [Appending to a File](#appending-to-a-file)
  4. [Deleting a File](#deleting-a-file)
  5. [Checking if Something is a File](#checking-if-something-is-a-file)
  6. [Checking if a File is Readable](#checking-if-a-file-is-readable)
  7. [Checking if a File is Writable](#checking-if-a-file-is-writable)
  8. [Copying a File](#copying-a-file)
  9. [Moving a File](#moving-a-file)
  10. [Getting a File's Directory Name](#getting-a-files-directory-name)
  11. [Getting a File's Basename](#getting-a-file-sbasename)
  12. [Getting a File's Name](#getting-a-files-name)
  13. [Getting a File's Extension](#getting-a-files-extension)
  14. [Getting a File's Size](#getting-a-files-size)
  15. [Getting a File's Last Modified Time](#getting-a-files-last-modified-time)
  16. [Getting Files in a Directory](#getting-files-in-a-directory)
  17. [Getting Files with Glob](#getting-files-with-glob)
  18. [Checking if a File or Directory Exists](#checking-if-a-file-or-directory-exists)
  19. [Creating a Directory](#creating-a-directory)
  20. [Deleting a Directory](#deleting-a-directory)
  21. [Getting the List of Directories](#getting-the-list-of-directories)
  22. [Copying a Directory](#copying-a-directory)

<a id="introduction"></a>
## Introduction
Most programs interact with a computer's file system in some way.  RDev comes with the `FileSystem` class to facilitate these interactions.  With it, you can easily read and write files, get attributes of files, copy files and folders, and recursively delete directories, and do other common tasks.

<a id="basic-usage"></a>
## Basic Usage
For all examples below, assume `$fileSystem = new \RDev\Files\FileSystem();`.

<a id="reading-a-file"></a>
#### Reading a File
```php
$fileSystem->read(FILE_PATH);
```

<a id="writing-to-a-file"></a>
#### Writing to a File
```php
// The third parameter is identical to PHP's file_put_contents() flags
$fileSystem->write(FILE_PATH, "foo", \LOCK_EX);
```

<a id="appending-to-a-file"></a>
#### Appending to a File
```php
$fileSystem->append(FILE_PATH, "foo");
```

<a id="deleting-a-file"></a>
#### Deleting a File
```php
$fileSystem->deleteFile(FILE_PATH);
```

<a id="checking-if-something-is-a-file"></a>
#### Checking if Something is a File
```php
$fileSystem->isFile(FILE_PATH);
```

<a id="checking-if-a-file-is-readable"></a>
#### Checking if a File is Readable
```php
$fileSystem->isReadable(FILE_PATH);
```

<a id="checking-if-a-file-is-writable"></a>
#### Checking if a File is Writable
```php
$fileSystem->isWritable(FILE_PATH);
```

<a id="copying-a-file"></a>
#### Copying a File
```php
$fileSystem->copy(SOURCE_FILE, TARGET_PATH);
```

<a id="moving-a-file"></a>
#### Moving a File
```php
// This is analogous to "cutting" the file
$fileSystem->move(SOURCE_FILE, TARGET_PATH);
```

<a id="getting-a-files-directory-name"></a>
#### Getting a File's Directory Name
```php
$fileSystem->getDirectoryName(FILE_PATH);
```

<a id="getting-a-files-basename"></a>
#### Getting a File's Basename
```php
// This returns everything in the file name except for the path preceding it
$fileSystem->getBaseName(FILE_PATH);
```

<a id="getting-a-files-name"></a>
#### Getting a File's Name
```php
// This returns the file name without the extension
$fileSystem->getFileName(FILE_PATH);
```

<a id="getting-a-files-extension"></a>
#### Getting a File's Extension
```php
$fileSystem->getExtension(FILE_PATH);
```

<a id="getting-a-files-size"></a>
#### Getting a File's Size
```php
// The size of the file in bytes
$fileSystem->getFileSize(FILE_PATH);
```

<a id="getting-a-files-last-modified-time"></a>
#### Getting a File's Last Modified Time
```php
$fileSystem->getLastModified(FILE_PATH);
```

<a id="getting-files-in-a-directory"></a>
#### Getting Files in a Directory
```php
// The second parameter determines whether or not we recurse into subdirectories
// This returns the full path of all the files found
$fileSystem->getFiles(DIRECTORY_PATH, true);
```

<a id="getting-files-with-glob"></a>
#### Getting Files with Glob
```php
// See documentation for PHP's glob() function
$filesSystem->glob($pattern, $flags);
```

<a id="checking-if-a-file-or-directory-exists"></a>
#### Checking if a File or Directory Exists
```php
$fileSystem->exists(FILE_PATH);
```

<a id="creating-a-directory"></a>
#### Creating a Directory
```php
// The second parameter is the chmod permissions
// The third parameter determines whether or not we create nested subdirectories
$fileSystem->createDirectory(DIRECTORY_PATH, 0777, true);
```

<a id="deleting-a-directory"></a>
#### Deleting a Directory
```php
// The second parameter determines whether or not we keep the directory structure
$fileSystem->deleteDirectory(DIRECTORY_PATH, false);
```

<a id="getting-the-list-of-directories"></a>
#### Getting the List of Directories
```php
// The second parameter determines whether or not we recurse into the directories
// This returns the full path of all the directories found
$fileSystem->getDirectories(DIRECTORY_PATH, true);
```

<a id="copying-a-directory"></a>
#### Copying a Directory
```php
$fileSystem->copyDirectory(SOURCE_DIRECTORY, TARGET_PATH);
```