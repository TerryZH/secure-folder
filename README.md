# secure-folder

A small macOS command-line tool that creates and opens an encrypted read/write folder backed by an encrypted macOS disk image.

## Why this design

This uses macOS `hdiutil`/DiskImages instead of a custom filesystem or custom encryption format. The secure folder is a normal mounted volume, so Finder and apps can read, write, rename, delete, and update files normally. When you close it, the encrypted content remains in the content container.

Default paths:

- Encrypted content container: `~/.secure-folder/content.sparsebundle`
- Mounted folder: `~/SecureFolder`

The default container is a `.sparsebundle`, which Finder presents as one item but is internally a bundle directory. It is preferred for long-lived sparse images. Use `--single-file` during setup if you require a physical single-file `.sparseimage`.

## Install

```bash
chmod +x secure-folder
sudo install -m 755 secure-folder /usr/local/bin/secure-folder
```

If `/usr/local/bin` is not on your PATH, use `~/bin` instead:

```bash
mkdir -p ~/bin
cp secure-folder ~/bin/secure-folder
chmod +x ~/bin/secure-folder
```

## Setup scenario

Interactive setup asks for the encrypted content container path, mount folder path, maximum size, and password:

```bash
secure-folder setup
```

Non-default example:

```bash
secure-folder setup \
  --content "$HOME/Documents/private.content.sparsebundle" \
  --mount "$HOME/Private" \
  --size 20g \
  --name Private
```

Single physical content file example:

```bash
secure-folder setup --single-file --content "$HOME/Documents/private.content.sparseimage"
```

## Everyday use scenario

Open and mount the secure folder:

```bash
secure-folder open
```

After entering the password, use `~/SecureFolder` like a normal folder.

Close and lock it:

```bash
secure-folder close
```

Check status:

```bash
secure-folder status
```

Reclaim unused space in the sparse container after deleting large files:

```bash
secure-folder compact
```

Change password:

```bash
secure-folder change-password
```

## Migration to another Mac

To use the same secure-folder data on another Mac, copy the encrypted content container. Do not copy the mounted folder.

Default container to copy:

```bash
~/.secure-folder/content.sparsebundle
```

Default mount folder that does not need to be copied:

```bash
~/SecureFolder
```

Before copying, close the secure folder so all data is flushed and the disk image is detached:

```bash
secure-folder close
```

Then copy the whole `.sparsebundle` package to the other Mac. A `.sparsebundle` looks like one item in Finder, but it is actually a directory bundle containing encrypted bands, so keep the entire bundle intact. Finder copy, `cp -a`, or `rsync -a` are safe choices:

```bash
mkdir -p ~/.secure-folder
cp -a /path/to/copied/content.sparsebundle ~/.secure-folder/
```

On the new Mac, install `secure-folder` and open it with the same password:

```bash
secure-folder open
```

If you used a custom content path, either put the container back at that same path or pass it explicitly:

```bash
secure-folder open --content "/path/to/content.sparsebundle" --mount "$HOME/SecureFolder"
```

If you created the container with `--single-file`, copy the `.sparseimage` file instead of a `.sparsebundle` directory.

## Security notes

- The tool never stores your password.
- Passwords are sent to `hdiutil` through stdin, not command-line arguments.
- Use a long passphrase. The script enforces 12 characters minimum; 16+ characters is better.
- While mounted, anyone with access to your logged-in macOS session can read the open folder. Close it when done.
- Keep a backup of the encrypted container. If you forget the password or the container is corrupted, the contents may be unrecoverable.
