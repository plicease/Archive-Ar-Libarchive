# Archive::Ar::Libarchive [![Build Status](https://secure.travis-ci.org/plicease/Archive-Ar-Libarchive.png)](http://travis-ci.org/plicease/Archive-Ar-Libarchive)

Interface for manipulating ar archives with libarchive

# SYNOPSIS

    use Archive::Ar::Libarchive;
    
    my $ar = Archive::Ar->new('libfoo.a');
    
    $ar->add_data('newfile.txt', 'some contents', { uid => 101, gid => 102 });
    
    $ar->add_files('./bar.tar.gz', 'bat.pl');
     
    $ar->remove('file1', 'file2');
    
    my $content = $ar->get_content('file3')->{data};
    
    my @files = $ar->list_files;
    
    $ar->write('libbar.a');
    
    my @file_list = $ar->list_files;

# DESCRIPTION

This module is a XS alternative to [Archive::Ar](https://metacpan.org/pod/Archive::Ar) that uses libarchive 
to read and write ar BSD, GNU and common ar archives.

There is no standard for the ar format.  Most modern archives are based 
on a common format with two extension variants, BSD and GNU.  Other 
esoteric variants (such as AIX (small), AIX (big) and Coherent) vary 
significantly from the common format and are not supported.  Debian's 
package format (.deb files) use the common format.

The interface attempts to be identical (with a couple of minor 
extensions) to [Archive::Ar](https://metacpan.org/pod/Archive::Ar) and the documentation presented here is 
based on that module. The diagnostic messages issued on error mostly 
come directly from libarchive, so they will likely not match exactly 
what [Archive::Ar](https://metacpan.org/pod/Archive::Ar) would produce, but it should issue a warning under
similar  circumstances.

The main advantage of [Archive::Ar](https://metacpan.org/pod/Archive::Ar) over this module is that it is 
written in pure perl, and thus does not require a compiler or 
libarchive.  The advantage of this module (at least as of this writing) 
is that it supports GNU (read) and BSD (read and write) extensions for 
longer member filenames.  As an XS module using libarchive it may also
be faster.

You may notice that the API to [Archive::Ar::Libarchive](https://metacpan.org/pod/Archive::Ar::Libarchive) and
[Archive::Ar](https://metacpan.org/pod/Archive::Ar) is similar to [Archive::Tar](https://metacpan.org/pod/Archive::Tar) and this was done
intentionally to keep similarity between the Archive::\* modules.

# METHODS

## new

    my $ar = Archive::Ar::Libarchive->new;
    my $ar = Archive::Ar::Libarchive->new($filename);
    my $ar = Archive::Ar::Libarchive->new($fh);

Returns a new [Archive::AR::Libarchive](https://metacpan.org/pod/Archive::AR::Libarchive) object.  Without a filename or 
glob, it returns an empty object.  If passed a filename as a scalar or a 
GLOB, it will attempt to populate from either of those sources.  If it 
fails, you will receive `undef`, instead of an object reference.

## set\_opt

    $ar->set_opt($name, $value);

Assign option `$name` value `$value`.  Supported options include:

- warn

    Warning level.  Levels are zero for no warnings, 1 for brief warnings,
    and 2 for warnings with a stack trace.  Default is zero.

- chmod

    Change the file permissions of files created when extracting.  Default
    is true (non-zero).

- same\_perms

    When setting file permissions, use the values in the archive unchanged.
    If false, removes setuid bits and applies the user's umask.  Default
    is true for the root user, false otherwise.

- chown

    Change the owners of extracted files, if possible.  Default is true.

- type

    Archive type.  May be GNU, BSD or COMMON, or undef if no archive
    has been read.  Defaults to the type of the archive read or `undef`.

    Note that libarchive can read GNU style ar files, but it cannot write
    to them.  If you attempt to write using [Archive::Ar::Libarchive](https://metacpan.org/pod/Archive::Ar::Libarchive)
    when type is set to GNU, it will throw an exception.

## get\_opt

    my $value = $ar->get_opt($name);

Returns the value of the option `$name`.

## type

    my $type = $ar->type;

Returns the type of the ar archive.  The type is undefined until an archive
is loaded.  If the archive displays characteristics of a gnu-style archive,
GNU is returned.  If it looks like a bsd-style archive, BSD is returned.
Otherwise, COMMON is returned.  Note that unless filenames exceed 16
characters in length, bsd archives look like the common format.

## clear

    $ar->clear;

Clears the current in-memory archive.

## read

    my $br = $ar->read($filename);
    my $br = $ar->read($fh);

This reads a new file into the object, removing any ar archive already
represented in the object.  The argument may be either a filename,
filehandle or IO::Handle object.  Returns the number of bytes read,
`undef` on failure.

## read\_memory

    my $br = $ar->read_memory($data);

This reads information from the first parameter, and attempts to parse 
and treat it like an ar archive. Like [Archive::Ar::Libarchive#read](https://metacpan.org/pod/Archive::Ar::Libarchive#read), 
it will wipe out whatever you have in the object and replace it with the 
contents of the new archive, even if it fails. Returns the number of 
bytes read (processed) if successful, `undef` otherwise.

## contains\_file

    my $bool = $ar->contains_file($filename)

Returns true if the archive contains a file with the name `$filename`.
Returns `undef` otherwise.

## extract

    $ar->extract;

Extract all files from the archive.  Extracted files are assigned the
permissions and modification time stored in the archive, and, if possible,
the user and group ownership.  Returns true on success, `undef` for failure.

## extract\_file

    $ar->extract_file($filename);

Extracts a single file from the archive.  The extracted file is assigned
the permissions and modification time stored in the archive, and, if
possible, the user and group ownership.  Returns true on success,
`undef` for faiure.

## rename

    $ar->rename($filename, $newname);

Changes the name of a file in the in-memory archive.

## chmod

TODO

## chown

TODO

## remove

    my $count = $ar->remove(@pathnames);
    my $count = $ar->remove(\@pathnames);

The remove method takes a filenames as a list or as an arrayref, and removes
them, one at a time, from the Archive::Ar object.  This returns the number
of files successfully removed from the archive.

## list\_files

    my @list = $ar->list_files;
    my $list = $ar->list_files;

This lists the files contained inside of the archive by filename, as
an array. If called in a scalar context, returns a reference to an
array.

## add\_files

    $ar->add_files(@filenames);
    $ar->add_files(\@filenames);

Takes an array or an arrayref of filenames to add to the ar archive,
in order. The filenames can be paths to files, in which case the path
information is stripped off. Filenames longer than 16 characters are
truncated when written to disk in the format, so keep that in mind
when adding files.

Due to the nature of the ar archive format, 
[Archive::Ar::Libarchive#add\_files](https://metacpan.org/pod/Archive::Ar::Libarchive#add_files) will store the uid, gid, mode, 
size, and creation date of the file as returned by 
[stat](https://metacpan.org/pod/perlfunc#stat).

returns the number of files successfully added, or `undef` on failure.

## add\_data

    my $size = $ar->add_data($filename, $data, $filedata);

Takes an filename and a set of data to represent it. Unlike 
[Archive::Ar::Libarchive#add\_files](https://metacpan.org/pod/Archive::Ar::Libarchive#add_files), 
[Archive::Ar::Libarchive#add\_data](https://metacpan.org/pod/Archive::Ar::Libarchive#add_data) is a virtual add, and does not 
require data on disk to be present. The data is a hash that looks like:

    $filedata = {
      uid  => $uid,   #defaults to zero
      gid  => $gid,   #defaults to zero
      date => $date,  #date in epoch seconds. Defaults to now.
      mode => $mode,  #defaults to 0100644;
    };

You cannot add\_data over another file however.  This returns the file 
length in bytes if it is successful, `undef` otherwise.

## write

    my $content = $ar->write;
    my $size = $ar->write($filename);

This method will return the data as an .ar archive, or will write to the 
filename present if specified. If given a filename, 
[Archive::Ar::Libarchive#write](https://metacpan.org/pod/Archive::Ar::Libarchive#write) will return the length of the file 
written, in bytes, or `undef` on failure. If the filename already exists, 
it will overwrite that file.

## get\_content

    my $hash = get_content($filename);

This returns a hash with the file content in it, including the data that the
file would naturally contain.  If the file does not exist or no filename is
given, this returns `undef`. On success, a hash is returned with the following
keys:

- name

    The file name

- date

    The file date (in epoch seconds)

- uid

    The uid of the file

- gid

    The gid of the file

- mode

    The mode permissions

- size

    The size (in bytes) of the file

- data

    The contained data

# get\_data

    my $data = $ar->get_data($filename);

Returns a scalar containing the file data of the given archive member.
On error returns `undef`.

## get\_handle

    my $handle = $ar->get_handle($filename);

Returns a file handle to the in-memory file data of the given archive
member.  On error returns `undef`.  This can be useful for unpacking
nested archives.

## error

    my $error_string = $ar->error($trace);

Returns the current error string, which is usually the last error
reported.  If a true value is provided, returns the error message
and stack trace.

# CAVEATS

libarchive cannot write GNU style ar files.  If you need to do that, you should
use [Archive::Ar](https://metacpan.org/pod/Archive::Ar) instead.

# SEE ALSO

- [Alien::Libarchive](https://metacpan.org/pod/Alien::Libarchive)
- [Archive::Ar](https://metacpan.org/pod/Archive::Ar)

# AUTHOR

Graham Ollis <plicease@cpan.org>

# COPYRIGHT AND LICENSE

This software is copyright (c) 2013 by Graham Ollis.

This is free software; you can redistribute it and/or modify it under
the same terms as the Perl 5 programming language system itself.

# POD ERRORS

Hey! **The above document had some coding errors, which are explained below:**

- Around line 112:

    You forgot a '=back' before '=head2'
