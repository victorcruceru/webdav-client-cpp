Webdavclient
===

Package Webdavclient provides easy and convenient to work with WebDAV-servers:

 - Yandex.Disk
 - Dropbox
 - Google Drive
 - Box
 - 4shared
 - ...

Install
===

```bash
> git clone https://github.com/designerror/webdavclient-cpp.git
> cd webdavclient-cpp
> clion .
```

Documentation
===

```bash
> cd docs
> doxygen doxygen.conf
> firefox html/index.html
```

Usage examples
===

```c++
#include <iostream>
#include <client.hpp>

int main()
{
  std::map<std::string, std::string> options =
  {
    {"webdav_hostname", "https://webdav.yandex.ru"},
    {"webdav_login", "{webdav_login}"},
    {"webdav_password", "{webdav_password}"}
  };
            
  WebDAV::Client client(options);
  
  auto check_connection = client.check();
  std::cout << "test connection with WebDAV drive is " 
            << check_connection ? "" : "not "
            << "successful"<< std::endl;
  
  auto is_directory = client.is_dir("/path/to/remote/resource");
  std::cout << "remote resource is " 
            << is_directory ? "" : "not " 
            << "directory" << std::endl;
  
  client.create_directory("/path/to/remote/directory/");
  client.clean("/path/to/remote/directory/");
  
  std::cout << "On WebDAV-disk available free space: " 
            << client.free_size() 
            << std::endl;
  
  std::cout << "remote_directory_name";
  for(auto resource_name : client.list("/path/to/remote/directory/"))
  {
    std::cout << "\t" << "-" << resource_name;
  }
  std::cout << std::endl;
  
  client.download("/path/to/remote/file", "/path/to/local/file");
  client.clean("/path/to/remote/file");
  client.upload("/path/to/remote/file", "/path/to/local/file");
  
  auto meta_info = client.info("/path/to/remote/resource");
  for(auto field : meta_info)
  {
    std::cout << field.first << ":" << "\t" << field.second;
  }
  std::cout << std::endl;
  
  client.copy("/path/to/remote/file1", "/path/to/remote/file2");
  client.move("/path/to/remote/file1", "/path/to/remote/file3");
  
  client.async_upload("/path/to/remote/file", "/path/to/local/file");
  client.async_download("/path/to/remote/file", "/path/to/local/file");
}
```

Acknowledgments
===
Thanks to the `JetBrains` company for

  [![Clion](http://s16.postimg.org/941c7ix9d/clion.png)](https://www.jetbrains.com/clion/)
