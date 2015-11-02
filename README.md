# proxy
Special Proxy Server which keeps as much in memory until the full file is downloaded

When we were using NGINX and a proxy server we noticed that sometimes we would have over 9 instances of the puller runnning for each REAL proxy we need.

To overcome this a new type of proxy is needed, this is it

In the real world we should only pull 100% of the source file, not 1000% so this proxy attempts to do this by becoming a multi part downloader.

On the 1st request we build up a map of each segment, 8mb per segment, and we start pulling the source from the real server and start building the file.

As we move from each 8mb segment we create a new one and save the old one to disk, see playback for the reload, and so we carry on until we get to the end or to the start of a different part that has only be downloaded.

When playing back from memory the playback system will have the block reloaded from disk and marked with a timeout value so we can dump it again.

So example
wget http://127.0.0.1:1234/fred.ts

will create a single download started pulling fred.ts to memory and this is what we get
   fred.ts  110 300000000000000000000000000000000000000000000000000000000000
   (map)        000000000000000000000000000000000000000000000000000000000000
   
Now the user decides they want to start playing from 30% into the file.
   fred.ts  220 300000000000000100000000000000000000000000000000000000000000
   (map)        000000000000000011111111111111111111111111111111111111111111

Now the user decides they want to start playing from 60% into the file.
   fred.ts  330 f70000000000000630000000000000000000000200000000000000000000
   (map)        000000000000000011111111111111111111111122222222222222222222
                AA             BB                      C

As you see we now have 3 downloads pulling in the source file, at 0% 30% 60%, and the map is marked so each will stop when the corresponding segment if finished.
AA is showing we have a 'f'ull segment which will be stored on disk after 45 seconds of no use, and 70% of the second segment.
BB shows we started downloading a partial block which is at 60% and the next one is at 30%
CC is the latest one at 20%


