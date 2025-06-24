# Resolving File System (RFS)

Resolving File System (RFS) is an experimental approach to interfacing resources
over network protocols, such as Gopher or HTTP/HTTPS via synthetic file system,
translating file system paths to URIs.

RFS works on the principal of no expectations towards servers.
The file system tries to resolve any path asked for by the user.
While it technically should not be problematic providing listings for available
resources, and it might be useful for some protocols, such as FTP/FTPS and SFTP,
many protocols, such as HTTP/HTTPS, do not provide any reliable way to list all
available resources.
This, and given the primary purpose of RFS is to provide an
interface for programs, RFS is not expected to provide resource listings.

Only resources needed to be listed are the '/:c/' and '/:config/' directories,
which are both equivalent in their contents.
'/:c/' and '/:config/' are used for interacting with the RFS server itself.
Implementations are free to put any files and directories they seem fit inside
'/:c/'.
Only file required in '/:c/' is '/:c/flush'.

RFS can hold cache of accessed resources.
'/:c/flush' can be used to flush this cache.
When resource is written to '/:c/flush', it is erased from cache.
If said resource does not exist or caching is not implemented, nothing happens.
When ':all' or ':a' is written to '/:c/flush', all cached
resources are flushed.
'/:c/flush' should ignore trailing newlines (hex 0A) and carriage returns
(hex 0D).

Request to any file outside '/:c/' and '/:config/' will be interpreted as a
resource. Resources are to be formed in the following format:

```
/[modifiers/]<resource>:
```

Such as:

```
/:nop/github.com/De-Alchmst/resolving-file-system-spec:
```

Note the colon at the end.
It is here due to technical limitations, aiding the file system to determine
whether a node is a directory or a regular file.

Modifiers are an arbitrary number of leading directories starting with colon.
They are used to provide extra information to the file system about
the specific request. Note that each request with any combination of modifiers
is considered a separate resource.
Only mandatory modifier to be implemented is ':nop', which does nothing, except
separating resources.

If resource does not exist, cannot be obtained, or contains invalid modifiers,
file I/O error should be returned.

Based on the protocol, RFS might implement writing.
If writing isn't implemented or specific resource cannot be written to, file
I/O error should be returned.

Upon successful write to a resource, response should be cached and be returned
upon the next, and only the next, read from the same resource done by the same
process, identified by it's PID.
These responses are separated from the optional regular cache and are not
affected by '/:c/flush'

When unsure, refer to the reference implementations, or just do whatever makes
sense for your protocol.
