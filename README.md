# Manage a motley crew of machines
I've got a handful of machines around the house, each of which does something
different. Some of them run [docker SSH
servers](https://github.com/spencertipping/docker), so I'd like to have my
`.ssh/config` updated automatically. `motley` is a bash DSL that makes it easy
to do host-specific configuration.

## Usage
First write a `Motfile`, which is a bash script that calls into motley
functions.

```bash
# Define the motley crew; this needs to happen first
machine foo alias vfoo:10.8.0.40
machine bar alias vbar:10.8.0.44
machine $EC2

# Add entries to ~/.bash_aliases to cd into these remote locations
# (You need https://github.com/spencertipping/cd for this to work)
foo sshfs f1:/mnt/vol1 f2:/mnt/vol2
bar sshfs b:/mnt/disk
$EC2 sshfs e:/home/me

# Machine-specific bash aliases
foo alias vpn zerovpn $vfoo -i ~/.ssh/vpn-key -p 2222 vpn@$EC2
bar alias vpn zerovpn $vbar -i ~/.ssh/vpn-key -p 2222 vpn@$EC2

# Tell foo how to build the spiff-o-matic:latest docker image
foo docker-source ~/spiff-o-matic spiff-o-matic:latest

# A bunch of containers from the image
for i in `seq 8`; do
  # Tell host "foo" how to spin up the container
  foo docker spiff$i spiff-o-matic:latest -p $((2421 + i)):22 -v /mnt

  # Add entries to ~/.ssh/config and ~/.bash_aliases for connection access
  foo ssh spiff$i -p $((2421 + i))              # alias spiff$i=...
  foo xpra Spiff$i :100 --no-keyboard-sync      # alias Spiff$i=...
done
```

Now use the motfile:

```sh
$ motley modules        # print all active modules
$ motley ls             # inspect all configurations
$ motley init           # configure local machine
$ motley start          # start all dockerized things, configure all machines
$ motley stop           # stop dockerized things, undo configurations
$ motley status         # check in with machines, print status
$ motley run            # do ongoing stuff; run this from a crontab
```
