            copyright: https://github.com/kuba--/zip

# `Examples`

Create a new zip archive with default compression level, and
Append directory files to the zip archive
```c
	zip_directory((char *)"dep_lib.zip", (char *)"dep_lib");
```	
	

	
Extract a zip archive into a folder.
```c
	int arg = 54;
	zip_extract_init_file("dep_lib.zip", "tmp", on_extract_entry, &arg);
```

Extract a zip archive into mem.
```c
	 zip_extract_init_mem((const void *)dep_lib_Buf, dep_lib_Size, TEMP_EXTRACT_DIR, on_extract_entry, &arg);
```


Create a new zip archive with default compression level.

```c
struct zip_t *zip = zip_open("foo.zip", ZIP_DEFAULT_COMPRESSION_LEVEL, 'w');
{
    zip_entry_open(zip, "foo-1.txt");
    {
        char *buf = "Some data here...";
        zip_entry_write(zip, buf, strlen(buf));
    }
    zip_entry_close(zip);

    zip_entry_open(zip, "foo-2.txt");
    {
        // merge 3 files into one entry and compress them on-the-fly.
        zip_entry_fwrite(zip, "foo-2.1.txt");
        zip_entry_fwrite(zip, "foo-2.2.txt");
        zip_entry_fwrite(zip, "foo-2.3.txt");
    }
    zip_entry_close(zip);
}
zip_close(zip);
```
Append to the existing zip archive.

```c
struct zip_t *zip = zip_open("foo.zip", ZIP_DEFAULT_COMPRESSION_LEVEL, 'a');
{
    zip_entry_open(zip, "foo-3.txt");
    {
        char *buf = "Append some data here...";
        zip_entry_write(zip, buf, strlen(buf));
    }
    zip_entry_close(zip);
}
zip_close(zip);
```

Extract a zip archive into a folder.

```c
int on_extract_entry(const char *filename, void *arg) {
    static int i = 0;
    int n = *(int *)arg;
    printf("Extracted: %s (%d of %d)\n", filename, ++i, n);

    return 0;
}

int arg = 2;
zip_extract("foo.zip", "/tmp", on_extract_entry, &arg);
Extract a zip entry into memory.
void *buf = NULL;
size_t bufsize;

struct zip_t *zip = zip_open("foo.zip", 0, 'r');
{
    zip_entry_open(zip, "foo-1.txt");
    {
        zip_entry_read(zip, &buf, &bufsize);
    }
    zip_entry_close(zip);
}
zip_close(zip);

free(buf);
```
Extract a zip entry into memory (no internal allocation).

```c
unsigned char *buf;
size_t bufsize;

struct zip_t *zip = zip_open("foo.zip", 0, 'r');
{
    zip_entry_open(zip, "foo-1.txt");
    {
        bufsize = zip_entry_size(zip);
        buf = calloc(sizeof(unsigned char), bufsize);

        zip_entry_noallocread(zip, (void *)buf, bufsize);
    }
    zip_entry_close(zip);
}
zip_close(zip);

free(buf);
```

Extract a zip entry into memory using callback.

```c
struct buffer_t {
    char *data;
    size_t size;
};

static size_t on_extract(void *arg, unsigned long long offset, const void *data, size_t size) {
    struct buffer_t *buf = (struct buffer_t *)arg;
    buf->data = realloc(buf->data, buf->size + size + 1);
    assert(NULL != buf->data);

    memcpy(&(buf->data[buf->size]), data, size);
    buf->size += size;
    buf->data[buf->size] = 0;

    return size;
}

struct buffer_t buf = {0};
struct zip_t *zip = zip_open("foo.zip", 0, 'r');
{
    zip_entry_open(zip, "foo-1.txt");
    {
        zip_entry_extract(zip, on_extract, &buf);
    }
    zip_entry_close(zip);
}
zip_close(zip);

free(buf.data);
```

Extract a zip entry into a file.

```c
struct zip_t *zip = zip_open("foo.zip", 0, 'r');
{
    zip_entry_open(zip, "foo-2.txt");
    {
        zip_entry_fread(zip, "foo-2.txt");
    }
    zip_entry_close(zip);
}
zip_close(zip);
```
List of all zip entries
```c
struct zip_t *zip = zip_open("foo.zip", 0, 'r');
int i, n = zip_total_entries(zip);
for (i = 0; i < n; ++i) {
    zip_entry_openbyindex(zip, i);
    {
        const char *name = zip_entry_name(zip);
        int isdir = zip_entry_isdir(zip);
        unsigned long long size = zip_entry_size(zip);
        unsigned int crc32 = zip_entry_crc32(zip);
    }
    zip_entry_close(zip);
}
zip_close(zip);
```
