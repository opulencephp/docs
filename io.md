# Input/Output

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
  11. [Getting a File's Basename](#getting-a-files-basename)
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

<h2 id="introduction">Introduction</h2>

Most programs interact with a computer's file system in some way.  Opulence comes with the `FileSystem` class to facilitate these interactions.  With it, you can easily read and write files, get attributes of files, copy files and folders, and recursively delete directories, and do other common tasks.

<h2 id="basic-usage">Basic Usage</h2>

For all examples below, assume `$fileSystem = new \Opulence\IO\FileSystem();`.

<h4 id="reading-a-file">Reading a File</h4>

```php
$fileSystem->read(FILE_PATH);
```

<h4 id="writing-to-a-file">Writing to a File</h4>

```php
// The third parameter is identical to PHP's file_put_contents() flags
$fileSystem->write(FILE_PATH, 'foo', \LOCK_EX);
```

<h4 id="appending-to-a-file">Appending to a File</h4>

```php
$fileSystem->append(FILE_PATH, 'foo');
```

<h4 id="deleting-a-file">Deleting a File</h4>

```php
$fileSystem->deleteFile(FILE_PATH);
```

<h4 id="checking-if-something-is-a-file">Checking if Something is a File</h4>

```php
$fileSystem->isFile(FILE_PATH);
```

<h4 id="checking-if-a-file-is-readable">Checking if a File is Readable</h4>

```php
$fileSystem->isReadable(FILE_PATH);
```

<h4 id="checking-if-a-file-is-writable">Checking if a File is Writable</h4>

```php
$fileSystem->isWritable(FILE_PATH);
```

<h4 id="copying-a-file">Copying a File</h4>

```php
$fileSystem->copy(SOURCE_FILE, TARGET_PATH);
```

<h4 id="moving-a-file">Moving a File</h4>

```php
// This is analogous to "cutting" the file
$fileSystem->move(SOURCE_FILE, TARGET_PATH);
```

<h4 id="getting-a-files-directory-name">Getting a File's Directory Name</h4>

```php
$fileSystem->getDirectoryName(FILE_PATH);
```

<h4 id="getting-a-files-basename">Getting a File's Basename</h4>

```php
// This returns everything in the file name except for the path preceding it
$fileSystem->getBaseName(FILE_PATH);
```

<h4 id="getting-a-files-name">Getting a File's Name</h4>

```php
// This returns the file name without the extension
$fileSystem->getFileName(FILE_PATH);
```

<h4 id="getting-a-files-extension">Getting a File's Extension</h4>

```php
$fileSystem->getExtension(FILE_PATH);
```

<h4 id="getting-a-files-size">Getting a File's Size</h4>

```php
// The size of the file in bytes
$fileSystem->getFileSize(FILE_PATH);
```

<h4 id="getting-a-files-last-modified-time">Getting a File's Last Modified Time</h4>

```php
$fileSystem->getLastModified(FILE_PATH);
```

<h4 id="getting-files-in-a-directory">Getting Files in a Directory</h4>

```php
// The second parameter determines whether or not we recurse into subdirectories
// This returns the full path of all the files found
$fileSystem->getFiles(DIRECTORY_PATH, true);
```

<h4 id="getting-files-with-glob">Getting Files with Glob</h4>

```php
// See documentation for PHP's glob() function
$filesSystem->glob($pattern, $flags);
```

<h4 id="checking-if-a-file-or-directory-exists">Checking if a File or Directory Exists</h4>

```php
$fileSystem->exists(FILE_PATH);
```

<h4 id="creating-a-directory">Creating a Directory</h4>

```php
// The second parameter is the chmod permissions
// The third parameter determines whether or not we create nested subdirectories
$fileSystem->createDirectory(DIRECTORY_PATH, 0777, true);
```

<h4 id="deleting-a-directory">Deleting a Directory</h4>

```php
// The second parameter determines whether or not we keep the directory structure
$fileSystem->deleteDirectory(DIRECTORY_PATH, false);
```

<h4 id="getting-the-list-of-directories">Getting the List of Directories</h4>

```php
// The second parameter determines whether or not we recurse into the directories
// This returns the full path of all the directories found
$fileSystem->getDirectories(DIRECTORY_PATH, true);
```

<h4 id="copying-a-directory">Copying a Directory</h4>

```php
$fileSystem->copyDirectory(SOURCE_DIRECTORY, TARGET_PATH);
```
