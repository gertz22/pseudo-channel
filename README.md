# PseudoChannel.py - Your Home-Brewed TV Channels

Joined by the author of [Fake TV](https://medium.com/@Fake.TV), this project aims at tackling one issue: creating a fake tv channel experience with your own media library (movies, tv shows, commercials, etc.). The idea is super simple... when you turn on your TV, rather than hopping straight to Netflix, you can choose to watch your own channel of curated media like a real channel, with randomized movie time blocks, weekend morning cartoons, 90's commercials to fill up gaps and more. We aim to add a ton of neat features but the basic idea is to have something that feels like a real TV channel. That being said it isn't supposed to "pause" nor are you supposed to intervene too much. Just like a real channel you are presented with a channel that you define once and let it go as it advances in series episodes, playing random movies where specified (defined by various parameters like genre, "Kevin Bacon", etc.). Think: weekday movie nights @ 8:00 PM. Or perhaps you want to further specify your weekly Wednesday evening movie be a movie in your Plex library that stars "Will Smith". PseudoChannel is built to interface with the Plex media server. So if you want to have your very own PseudoChannel, you first need to set up your home server using [Plex](https://www.plex.tv/). After that you can come back here to learn how to setup everything else. Although this app runs using Python and the command line, we aim to make all of it as easy as possible to understand for those who are intimidated by this sort of technology. If you have a question, are troubleshooting or have feature ideas, just leave an 'issue' here in this repository.


![Generated HTML schedule](http://i.imgur.com/uTGRYIp.png)

## How to Use:

- The instructions below are all for configuring the **"controller"** device (i.e. a laptop or raspberry pi running linux). This is the device this app runs on to control the Plex client. The **"client"** device should be a Raspberry Pi running Rasplex hooked up to your TV via HDMI - although I'm sure other devices work great too (never tried). 

1. PseudoChannel uses Python 2.7. The recommended method of setting up most python environments is to use [virtualenv](http://python-guide-pt-br.readthedocs.io/en/latest/dev/virtualenvs/). This keeps all your pip packages / python versions seperated per project basis. I find this method extremely useful but also somewhat unintuitive, especially at first. Whether you choose to use "virtualenv" to isolate your project environment or not, you can install all the PseudoChannel.py dependencies by running the following command after downloading this repository:

```bash
% pip install -r requirements.txt
```
*You need to run the previous command using `sudo` if not in a virtualenv.*

2. In order to tell PseudoChannel.py how to connect to your Plex server, create an empty file named, `plex_token.py` just outside of the project directory. Within that file add your plex server url / [plex token](https://support.plex.tv/hc/en-us/articles/204059436-Finding-an-authentication-token-X-Plex-Token) like so:

```bash
token = '<your token>'
baseurl = 'http://192.168.1.28:32400'
```
*This file is important as it tells PseudoChannel.py how/where to connect to your Plex server. It should sit just outside of this /pseudo-channel/ directory.*

3. Edit the `pseudo_config.py` / `the pseudo_schedule.xml` to your liking. You can specify your plex media library names within the `pseudo_config.py` file... If you do not intend on using commercials just set the `useCommercialInjection` flag to `False`. There are a few other experimental options like using Google Calendar rather than an XML. It is an arduous process to initially set up and I've found the XML method to be the easiest method for organizing your schedule - so stick with that for now. Finally, setup your schedule in the xml file. There are some detailed instructions commented at the top of that file.

4. Run the `PseudoChannel.py` file with the following flags:

```bash
% python PseudoChannel.py -u -xml -g -m -r
```
*You can also run `-h` to view all the options. Use `-c` to find the name(s) of your Plex client(s) to add to the config.*

- The `-u` flag will prepare & update (& create if not exists) the local `pseudo-channel.db`, you only need to run this once in the beginning or later when you have added new media to your Plex libraries. 
- The `-xml` flag will update the newly created local db with your schedule from the xml file - you  should run this everytime you make changes to the xml. 
- The `-g` file will generate the daily schedule (for today) based on the xml. This is useful for the first run or testing (or manually advancing the daily queue forward). Running this flag say, 15 times will advance the play queue forward 15 days. It is automatically run every night at midnight to generate the daily schedule.
- The `-m` flag makes both the .html/.xml files and starts a simple html web server in the `./schedules` directory.
- Finally, the `-r` flag will run the app, checking the time / triggering the playstate of any media that is scheduled. It will also update the daily schedule when the clock hits 11.59 (or whatever time you've configured in the config file). If you see errors, check your entries in the xml first. Check your times, check for overlaps & make sure your are using ascii characters to replace foreign characters like umlauts and '&' characters, etc. Make sure all of your movie names / TV Series names are correct. 

You can run `% python PseudoChannel.py` with the following options. The order is important depending on what you are doing (i.e. `% python PseudoChannel.py -u -xml -g -m -r`):

| Flag                    | Description   | 
| ------------------------|--------------| 
| -u, --update            | Manually update (or create if not exists) the local db when new media is added to your Plex server. |
| -xml, --xml             | After making any edits your .xml schedule. Run this to populate the local db. |  
| -g, --generate_schedule | Manually generate the daily schedule. This is useful for testing / first run. |
| -r, --run               | Run PsuedoChannel.py. |
| -c, --show_clients      | Show connected Plex clients. |
| -s, --show_schedule     | Output the generated "Daily Schedule" to your terminal. |
| -m, --make_html         | Manually generate both html / xml docs based on the "Daily Schedule". |
| -e, --export            | Export the current queue of your "TV Shows" episodes. Useful when redoing your local DB. |
| -i, --import            | Import the previously exported queue of your "TV Shows" episodes. |

## `startstop.sh` - Alternative Way of Running the Application:

Within the app directory, there is a file named, `startstop.sh`. This bash script is a convenient way to start/stop the application. When run, it will start the application and save the "pid" of the process to the application directory. When run again, it will check to see if that process is running, if so it will stop it. All you have to do is run:

```bash
% ./startstop.sh
```

When you start the application with this bash script, you can close your terminal as it will keep running in the background. Later, when you come back and want to stop it... you can just execute that file once more and it will stop the running process. Please note: It's good to test the application and your configurations using the manual process above before running this bash script. Although there is a `pseudo-channel.log` that is created within the application directory, it is easier to just view the output in your terminal window - something that won't happen when using the bash script. 

## Futher Info:

Features are being added to the xml but as of now there are a few. Within the XML `<time>` entry you are able to pass in various attributes to set certain values. As of now, aside from "title" and "type" which are mandatory, you can take advantage of "time-shift". This parameter accepts values in minutes and can be no lower than "1". If the attribute, "strict-time" is set to "false", then this `<time>` entry will be shifted to a new time based on the previous time with a smaller gap calculated according to the value in "time-shift". Basically, if you do not want any gaps in your daily generated schedule you would leave "strict-time" false and set "time-shift" to "1" for all `<time>` entries. However, this will create a schedule with weird start times like, "1:57 PM". Taking advantage of the "time-shift" perameter will correct this. If you set it to a value of "5", all media is shifted and hooked on to a "pretty" time that is a multiple of 5. So if used, rather then having a "Seinfeld" episode being set to "1:57 PM" it may be recalculated and scheduled for "2:00 PM". However, if you would like to make sure that "The Simpsons" will always start every weekday at "6:00 PM" then you can simply set that `<time>` entry to `srtict-time="true"`. This will ensure that despite other non-strict times shifting around, "The Simpsons" will air every weekday at the desired "6:00 PM" as scheduled (be sure that you haven't accidentally made two time entries "strict-time" for the same day/time - this sort of thing will cause weird scheduling errors). When using "strict-time" or having the "time-shift" value > than 1 (minute), this will result in empty gaps in the schedule. Currently I have a flag in the config for "commercial injection" to fill up the gaps as much as possible with commercials from commercials in your Plex server "Commercials" library. If you do not want to use this feature or if you don't have any commercials in your Plex server, just open up that `pseudo_config.py` file and set `useCommercialInjection` to `False`. Please check the `pseudo-channel.xml` for more information.

## The Automatically Generated .HTML Daily Schedule / Server

To view the automatically generated "Daily Pseudo Schedule" index.html as seen in the image above, find it in the generated `./schedules/` directory within the project folder. The html file is generated both when the daily schedule is updated and whenever a media item from the schedule plays or ends (you can manually generate it with the `-m` flag). 

If you configured your `controllerServerPath` variable in the `pseudo_config.py` file, you can view your schedule by pointing your browser here:

```bash
http://192.168.1.28:8000
```
*Where `192.168.1.28` is the IP of your controller & `8000` is the port - both perameters are configured in the `pseudo_config.py` file.*

Stay tuned for a polished version / bug fixes. I've also started a user friendly web version that hopefully will be working soon.  




