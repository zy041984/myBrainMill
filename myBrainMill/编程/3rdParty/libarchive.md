3.8.1
# 编译
cmake选择了enable_OpenSSL和Enable_ZSTD
修改了cmakeList.txt找不到gcc的两个message(FATAL_ERROR)，把这两行注释了
# 使用
## 归档
1) Ask `archive_write_new` for an archive writer object.
2) Set any global properties.  In particular, you should set the compression and format to use.
3) Call `archive_write_open` to open the file (most people will use `archive_write_open_file` or `archive_write_open_fd`, which provide convenient canned I/O callbacks for you).
4) For each entry:
 - construct an appropriate struct `archive_entry` structure
 - `archive_write_header` to write the header
 - `archive_write_data` to write the entry data
5) `archive_write_close` to close the output
6) `archive_write_free` to cleanup the writer and release resources
## 读档
1) Ask `archive_read_new` for an archive reader object.
2) Update any global properties as appropriate. In particular, you'll certainly want to call appropriate
 `archive_read_support_XXX` functions.
3) Call `archive_read_open_XXX` to open the archive
4) Repeatedly call `archive_read_next_header` to get information about successive archive entries. Call `archive_read_data` to extract data for entries of interest.
5) Call archive_read_free to end processing.

## 归档 
中文路径 zstd压缩 保留文件的时间隐藏只读属性
```
//把一个文件夹下面所有内容归档为一个文件，并用zstd压缩
//输入参数const wchar_t* dirPath    被归档的文件夹
//输入参数const wchar_t* outputPath 输出文件的绝对路径名
//输入参数unsigned long long& totalSize 被归档的文件夹的大小
//返回值，成功返回ARCHIVE_OK=0，其他情况返回非零值
int compressDirToZstd(const wchar_t* dirPath, const wchar_t* outputPath, unsigned long long& totalSize)
{
    struct archive* a = archive_write_new();
int ret = archive_write_add_filter_zstd(a);
if (ret != ARCHIVE_OK)
{
    archive_write_free(a);
    return ret;
}
//ret = archive_write_set_format_ustar(a);//旧POSIX格式，大小和文件名长度受限
ret = archive_write_set_format_pax_restricted(a); // 使用PAX格式保留元数据
if (ret != ARCHIVE_OK)
{
    archive_write_free(a);
    return ret;
}
ret = archive_write_set_format_option(a, "", "hdrcharset", "UTF-8");
if (ret != ARCHIVE_OK)
{
    archive_write_free(a);
    return ret;
}
ret = archive_write_open_filename_w(a, outputPath);
if (ret != ARCHIVE_OK)
{
    archive_write_free(a);
    return ret;
}
size_t usedSize{ 0 };
for (const auto& entry : std::filesystem::recursive_directory_iterator(dirPath)) {
    //if (entry.is_directory()) continue;
    if (m_stop.load())
        break;
    struct archive_entry* ae = archive_entry_new();
    std::wstring relPathW = entry.path().wstring().substr(std::filesystem::path(dirPath).wstring().length() + 1);
    //std::wcout << entry.path().wstring().c_str() << std::endl;
    std::string utf8 = WStringToUTF8(relPathW);
    archive_entry_set_pathname_utf8(ae, utf8.c_str());
    //std::string relPath = entry.path().string().substr(fs::path(dirPath).wstring().length() + 1);
    //archive_entry_set_pathname(ae, relPathW.c_str());
    //archive_entry_set_size(ae, entry.file_size());
    //archive_entry_set_filetype(ae, AE_IFREG);
    //archive_entry_set_perm(ae, 0644);
    struct _stat64 filestatus;
    int ret = _wstat64(entry.path().wstring().c_str(), &filestatus);
    if (ret == 0)
    {
        archive_entry_set_atime(ae, filestatus.st_atime, 0);//上次访问时间
        archive_entry_set_mtime(ae, filestatus.st_mtime, 0);//上次修改时间
        archive_entry_set_birthtime(ae, filestatus.st_ctime, 0);//创建时间
        archive_entry_set_mode(ae, filestatus.st_mode);//文件模式
        //std::cout << "st_mode 0x" << std::hex << filestatus.st_mode << std::dec << std::endl;
        if (filestatus.st_mode & _S_IFMT)// File type mask
        {
            //std::cout << "_S_IFMT" << std::endl;
        }
        if (filestatus.st_mode & _S_IFDIR)// Directory
        {
            //std::cout << "_S_IFDIR" << std::endl;
        }
        if (filestatus.st_mode & _S_IFCHR)// Character special
        {
            //std::cout << "_S_IFCHR" << std::endl;
        }
        if (filestatus.st_mode & _S_IFIFO)// Pipe
        {
            //std::cout << "_S_IFIFO" << std::endl;
        }
        if (filestatus.st_mode & _S_IFREG)// Regular
        {
            archive_entry_set_size(ae, entry.file_size());
            archive_entry_set_filetype(ae, AE_IFREG);
            //std::cout << "_S_IFREG" << std::endl;
        }
        if (filestatus.st_mode & _S_IREAD)// Read permission, owner
        {
            //std::cout << "_S_IREAD" << std::endl;
        }
        if (filestatus.st_mode & _S_IWRITE)// Write permission, owner
        {
            //std::cout << "_S_IWRITE" << std::endl;
        }
        if (filestatus.st_mode & _S_IEXEC)// Execute/search permission, owner
        {
            //std::cout << "_S_IEXEC" << std::endl;
        }
        DWORD attributes = GetFileAttributes(entry.path().wstring().c_str());
        if (attributes == INVALID_FILE_ATTRIBUTES) {
            //std::cerr << "Error: " << GetLastError() << std::endl;
        }
        else {
            if (attributes & FILE_ATTRIBUTE_DIRECTORY) {
                //std::cout << " is a directory." << std::endl;
            }
            else {
                //std::cout << " is a file." << std::endl;
            }
            //bool hidden{ false };
            //bool system{ false };
            //bool rdonly{ false };
            std::wstring flagStr;
            //unsigned long flag{ 0 };
            if (attributes & FILE_ATTRIBUTE_HIDDEN) {
                //std::cout << " is hidden." << std::endl;
                //hidden = true;
                flagStr += std::wstring(L"hidden");
                //flag |= FILE_ATTRIBUTE_HIDDEN;
                //archive_entry_set_fflags(ae, FILE_ATTRIBUTE_HIDDEN,0);//文件模式
               // archive_entry_copy_fflags_text_w(ae, L"hidden");
            }
            if (attributes & FILE_ATTRIBUTE_SYSTEM) {
                //std::cout << " is system." << std::endl;
                //system = true;
                if (!flagStr.empty())
                    flagStr += std::wstring(L",");
                flagStr += std::wstring(L"system");
                //flag |= FILE_ATTRIBUTE_SYSTEM;
                //archive_entry_set_fflags(ae, FILE_ATTRIBUTE_SYSTEM, 0);
                //archive_entry_copy_fflags_text_w(ae, L"system");
            }
            if (attributes & FILE_ATTRIBUTE_READONLY) {
                //rdonly = true;
                if (!flagStr.empty())
                    flagStr += std::wstring(L",");
                flagStr += std::wstring(L"rdonly");
                //flag |= FILE_ATTRIBUTE_READONLY;
                //std::cout << " is readonly." << std::endl;
                //archive_entry_set_fflags(ae, FILE_ATTRIBUTE_READONLY, 0);
                //archive_entry_copy_fflags_text_w(ae, L"rdonly");
            }
            if (!flagStr.empty())
                archive_entry_copy_fflags_text_w(ae, flagStr.c_str());
        }
    }
    ret=archive_write_header(a, ae);
    if (ret != ARCHIVE_OK)
        continue;
    if (filestatus.st_mode & _S_IFREG)
    {
        HANDLE hFile = CreateFileW(entry.path().wstring().c_str(),
            GENERIC_READ,
            FILE_SHARE_READ,
            NULL,
            OPEN_EXISTING,
            FILE_ATTRIBUTE_NORMAL,
            NULL);
        if (hFile != INVALID_HANDLE_VALUE) {
            const size_t bufSize = 1024 * 1024;
            char* buffer = new char[bufSize];
            DWORD bytesRead;
            while (ReadFile(hFile, buffer, bufSize, &bytesRead, NULL) && bytesRead > 0) {
                archive_write_data(a, buffer, bytesRead);
                usedSize += bytesRead;
                float prog = static_cast<float>(usedSize) / totalSize;
                emit reportArchiveProgress(mRole, prog);
            }
            delete[] buffer;
            CloseHandle(hFile);
        }
    }
    archive_entry_free(ae);
}
ret = archive_write_close(a);
if (ret != ARCHIVE_OK)
{
    archive_write_free(a);
    return ret;
}
ret = archive_write_free(a);
    return ARCHIVE_OK;//忽略最后一个archive_write_free的失败
}
```
## 解压缩 
中文路径 zstd解压缩 保留文件的时间隐藏只读属性
```
//把一个zstd压缩的归档文件还原到文件夹
//输入参数const wchar_t* archivePath    压缩文件的绝对路径名
//输入参数const wchar_t* outputDir      还原到这个文件夹
//返回值，成功返回ARCHIVE_OK=0，其他情况返回非零值
int extractWithMetadata(const std::wstring& archivePath, const std::wstring& outputDir)
{
// 初始化libarchive对象
struct archive* a = archive_read_new();
int ret=archive_read_support_format_all(a);
if (ret != ARCHIVE_OK)
{
    archive_read_free(a);
    return ret;
}
ret = archive_read_support_filter_zstd(a);
if (ret != ARCHIVE_OK)
{
    archive_read_free(a);
    return ret;
}
ret = archive_read_set_options(a, "hdrcharset=UTF-8");
if (ret != ARCHIVE_OK)
{
    archive_read_free(a);
    return ret;
}
// 打开归档文件
if (archive_read_open_filename_w(a, archivePath.c_str(), 10240) != ARCHIVE_OK)
{
    //std::wcerr << L"无法打开归档文件: " << archivePath << std::endl;
    archive_read_free(a);
    return -1;
}

struct archive_entry* entry;
while (archive_read_next_header(a, &entry) == ARCHIVE_OK && !m_stop.load())
{
    //struct archive_entry* ae = archive_entry_new();
    const wchar_t* wpath = archive_entry_pathname_w(entry);
    if (!wpath) {
        //应该不会走入这个分支，查看归档后的zst文件，已经能看到文件名，
        // 说明libarchive保存文件的时候已经转化为当前编码页
        const char* utf8path = archive_entry_pathname_utf8(entry);
        if (utf8path) {
            int size = MultiByteToWideChar(CP_UTF8, 0, utf8path, -1, NULL, 0);
            wchar_t* buffer = new wchar_t[size];
            MultiByteToWideChar(CP_UTF8, 0, utf8path, -1, buffer, size);
            archive_entry_copy_pathname_w(entry, buffer);
            delete[] buffer;
        }
    }

    std::filesystem::path fullPath = std::filesystem::path(outputDir) / archive_entry_pathname_w(entry);
    archive_entry_set_pathname_utf8(entry, reinterpret_cast<const char*>(fullPath.u8string().c_str()));

    // 提取文件
    int flags = ARCHIVE_EXTRACT_TIME | ARCHIVE_EXTRACT_PERM | ARCHIVE_EXTRACT_ACL;
    if (archive_read_extract(a, entry, flags) != ARCHIVE_OK)
    {
        //std::wcerr << L"提取失败: " << fullPath << std::endl;
        continue;
    }
    // 设置Windows文件属性
    DWORD attrs = 0;
    unsigned short mode = archive_entry_mode(entry);
    if (mode & _S_IFMT)
    {
        //archive_entry_set_fflags(entry, FILE_ATTRIBUTE_HIDDEN, 0);
        unsigned long set{ 0 }, clear{ 0 };
        archive_entry_fflags(entry, &set, &clear);
        const char* tmp = archive_entry_fflags_text(entry);
        if (tmp)
        {
            std::string flagStr(tmp);
            //std::cout << "flagStr " << flagStr.c_str() << std::endl;
            bool hasHidden = (flagStr.find("hidden") != std::string::npos);
            bool hasRdonly = (flagStr.find("rdonly") != std::string::npos);
            bool hasSystem = (flagStr.find("system") != std::string::npos);
            if (hasHidden)
            {    //std::cout << "flagStr " << flagStr.c_str() << std::endl;
                attrs |= FILE_ATTRIBUTE_HIDDEN;
                //std::cout << "hidden" << std::endl;
            }
            if (hasRdonly)
            {    //std::cout << "flagStr " << flagStr.c_str() << std::endl;
                attrs |= FILE_ATTRIBUTE_READONLY;
                //std::cout << "rdonly" << std::endl;
            }
            if (hasSystem)
            {    //std::cout << "flagStr " << flagStr.c_str() << std::endl;
                attrs |= FILE_ATTRIBUTE_SYSTEM;
                //std::cout << "system" << std::endl;
            }
        }
    }
    // 应用文件属性
    if (attrs != 0) {
        SetFileAttributesW(fullPath.c_str(), attrs);
    }

    HANDLE hFile = CreateFileW(fullPath.c_str(), FILE_WRITE_ATTRIBUTES,
        FILE_SHARE_READ | FILE_SHARE_WRITE,
        NULL, OPEN_EXISTING,
        FILE_FLAG_BACKUP_SEMANTICS, NULL);
    if (hFile != INVALID_HANDLE_VALUE)
    {
        FILETIME mtime, atime, ctime;
        time_t mtime_t = archive_entry_mtime(entry);
        time_t atime_t = archive_entry_atime(entry);
        time_t btime_t = archive_entry_birthtime(entry);

        // 转换时间格式
        LONGLONG llValue = Int32x32To64(mtime_t, 10000000) + 116444736000000000;
        mtime.dwLowDateTime = (DWORD)llValue;
        mtime.dwHighDateTime = llValue >> 32;

        llValue = Int32x32To64(atime_t, 10000000) + 116444736000000000;
        atime.dwLowDateTime = (DWORD)llValue;
        atime.dwHighDateTime = llValue >> 32;

        llValue = Int32x32To64(btime_t, 10000000) + 116444736000000000;
        ctime.dwLowDateTime = (DWORD)llValue;
        ctime.dwHighDateTime = llValue >> 32;

        SetFileTime(hFile, &ctime, &atime, &mtime);
        CloseHandle(hFile);
    }
    //archive_entry_free(entry);
}
ret=archive_read_close(a);
if (ret != ARCHIVE_OK)
{
    archive_read_free(a);
    return ret;
}
ret = archive_read_free(a);
    return ARCHIVE_OK;//忽略最后一个archive_read_free的失败
}
```