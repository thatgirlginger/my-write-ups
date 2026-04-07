this one was actually pretty easy!

(all run on kali linux VM)

i put on a lot of ctf walkthroughs/pentesting youtube videos as background noise, and i remembered the zsteg tool being used a few times

i searched around a bit to find installation info. from what i gather, because the library is in ruby, to install on kali linux the first command is

first, of course
```
sudo apt install ruby
```
```
sudo gem install zsteg
```

now all you have to do is run
```
zsteg pico.flag.png
```

and there you have it!
