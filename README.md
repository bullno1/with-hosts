# with-hosts -- Override `hosts` file for a single process

## Example

```sh
git clone https://github.com/bullno1/with-hosts.git
cd with-hosts

echo "127.0.0.1 mywebsite.com www.mywebsite.com" >> hosts

# Start a local webserver
python -m http.server &

./with-hosts hosts curl http://mywebsite.com:8000

# You should see the content of the current folder
```

## Why?

When testing websites and ~~cloud~~ butt clients, I often find myself having to edit the global hosts file.
With multiple projects, it quickly gets out of hand.
Moreover, those settings can't be checked into version control.
With this utility, one can test clients easily without having to maintain multiple configurations/profiles.

## Building

Run `make` in the project folder and a file called `libhostspriv.so` will be created.

## How it works

`libhostspriv.so` is force loaded using `LD_PRELOAD`.
It overrides several calls in libc related to host name lookup.
The library looks into the file pointed to by the `HOSTS_FILE` environment variable before passing control to the original libc functions.

`with-hosts` is a shell script that helps setting up the environment variables and even automatically compile `libhostspriv.so` if it is not present.
This project is intended to be dropped into your project (via git submodule, for example) and just works without any complex setups.

## Usage

```sh
with-hosts <hosts-file> [cmd [args]]
```

If `cmd` is not specified, it will open a new shell (using the `SHELL` environment variable) with `hosts` file overidden.

The `hosts` file has the following syntax:

```txt
# Characters after # is ignored
# IP-address Host-names
127.0.0.1 myhost.com myhostalias.com
172.28.128.3 my-vm.com
```

## Related projects

This project was forked from https://github.com/figiel/hosts.
However, instead of using the `HOME` variable to point to the directory containing the `hosts` file,
I decided to use the `HOSTS_FILE` variable to point directly to the `hosts` file.
This is because of two reasons:

- Many applications rely on `HOME` for their settings, changing it can have unexpected consequences.
- Pointing directly to the `hosts` file gives more flexibility.
  For example: `hosts` file does not have to be named "hosts".

## Limitations

Anything that doesn't rely on libc to resolve host names will not work.
Does not work for suid programs either.
