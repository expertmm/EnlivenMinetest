# minetest-chunkymap
A Minetest online web live map generator not requiring mods, with emphasis on compatibility without regard to speed, efficiency, or ease of use.

Compatible with GNU/Linux systems, Windows, or possibly other systems (but on Windows, chunkymap-regen.py must be scheduled by hand with Scheduled Tasks)

License: (see LICENSE in notepad or your favorite text editor)
This program comes without any warranty, to the extent permitted by applicable law.

## Features:
* Runs as python script (loop by default to reduce disc reads since stores certain info) run like:
    python chunkymap-regen.py
	or to get back to it later with screen -r, instead install screen command (or tmux may work) then run:
	screen -t chunkymapregen python $HOME/chunkymap/chunkymap-regen.py
	#where -t chunkymapregen just names the screen chunkymapregen
	#Then if you are using screen and want to leave the output without terminating the process press Ctrl a d
	#NOTE: now that loop is default, cron job scripts, which now disable loop for compatibility with new version, are ALL optional and NOT recommended
    # ( to run only once, run: python chunkymap-regen.py --no-loop true )
* Change program options (or stop it) while looping or rendering by placing chunkymap-signals.txt in the same directory as chunkymap-regen.py (see chunkymap-signals example files)
	* to maintain stability of  your text editor, save the file, close it, then move/copy it to the directory (or save it as something else then rename it to chunkymap-signals.txt).
	* alternatively, in *nix do something like:
    echo "refresh_map_enable:False" > $HOME/chunkymap/chunkymap-signals.txt
	sleep 15s
	echo "loop_enable:False" > $HOME/chunkymap/chunkymap-signals.txt
	* list of signals:
		loop_enable:True
		loop_enable:False
		#verbose_enable is false for looped (default) mode and true for non-looped mode
		verbose_enable:True
		verbose_enable:False
		refresh_players_enable:True
		refresh_players_enable:False
		refresh_map_enable:True
		refresh_map_enable:False
		#rerenders chunks that were rendered in this run:
		recheck_rendered:True
		#where 1 is number of seconds (only delays first iteration--further iterations continue until refreshing player is needed):
		refresh_map_seconds:1
		#where 1 is number of seconds:
		refresh_players_seconds:1
* Save jpg or png named as player's index to the players folder of the world to change player's icon on map (index is a number assigned for use with ajax when $show_player_names_enable is false). The index can be found in the player's yml file generated by chunkymap-regen.py.
		
* Can show a static html version of map (echo_chunkymap_table() php function) -- see viewchunkymap.php
	* Zoom in and out
	* optionally echo name of world that was detected by the scheduled py file
	* shows player location (can optionally show only first characters of name, for privacy; there is no saved setting yet, so to adjust, you must change the value of $nonprivate_name_beginning_char_count in chunkymap.php)	
	* Ghost players if they stay in one spot long enough (see $player_file_age_idle_max_seconds in chunkymap.php)
	* Hide players if they stay in one spot long enough (see $player_file_age_expired_max_seconds in chunkymap.php) avoiding logout detection, and not requiring mods
* Has optional script to add crontab entry (to schedule update script every minute that runs the py file unless the py file is not complete [took longer than 1 minute])

## Developer Notes:
* Player username privacy: check_players in chunkymap-regen.py intentionally makes up an index and uses that as the filename on the destination, so that ajax can update players without knowing either their id (filename of minetest player file) or display name (listed in the player file)
(this way, only usernames can be known if chunkymap.php allows that, or the person is logged in to the server)
Because of the feature, chunkymap-regen.py must prevent duplicates based on value of id in the resulting yml files (minetest player filename as id).
This should be hard to corrupt since id is used as the indexer for the players dict (however, extra files with no matching entry in the dict will still need to be deleted if they exist)
* games_path, mods_path, players_path, and other subdirectories of the major ones should not be stored in minetestmeta.yml, since otherwise the values may deviate from the parent directories when the parent directories change.
To avoid this problem, instead derive the paths from the parent paths using your favorite language such as in the following examples:
	games_path = os.path.join(minetestinfo.get_var("shared_minetest_path"), "games")
	mods_path = os.path.join(minetestinfo.get_var("game_path"), "mods")
	players_path = os.path.join(minetestinfo.get_var("primary_world_path"), "players")
	world_path = None
	world_name = None
	if minetestinfo.contains("primary_world_path"):
		world_path = minetestinfo.get_var("primary_world_path")
		world_name = os.path.basename(world_path)
* Keep in mind that gameid (in game.conf in a subgame folder, and world.mt in a world folder) is NOT case-sensitive: for example, minetest_game has the gameid 'Minetest' (first letter capitalized) but the worlds generated by Minetest client have the gameid 'minetest' (lowercase) in their world.mt
	Yet somehow for everything else, gameid in world.mt is the name of the game FOLDER (NOT the name variable in the folder's game.conf)
* the map update function is only able to detect new chunks, and only checks edge chunks if player is present in one
* The following are saved to chunkymap.yml if not already set:
www_minetest_path (such as /var/www/html/minetest)
user_minetest_path
world_name
world_path

## Requirements:
* A minetest version compatible with minetestmapper-numpy.py Made by Jogge, modified by celeron55
* Python 2.7 (any 2.7.x)
* Other requirements for Windows are below; other requirements for GNU/Linux are flock command (only if you schedule the chunkymap-cronjob script), and anything installed by install-chunkymap-on-ubuntu.sh (for other distros, modify it and send me a copy as a GitHub issue as described below in the Installation section)

## Installation
(NOTE: map refresh skips existing tiles unless you delete the related png and text files in your chunkymapdata directory)
* If you are not using Ubuntu, first edit the installer for your distro (and please send the modified file to me [submit as new issue named such as: DISTRONAME installer except instead of DISTRONAME put the distro you made work])
* If you are using Ubuntu
	* Install the git version of minetest (or otherwise install 0.4.13 or other version compatible with the map generators used by chunkymap)
	OPTION 2: IF you are using Ubuntu go to a terminal, cd to this directory,  
	then switch user to the one that will run minetestserver
	(since install-chunkymap-on-ubuntu.sh DOES replace "/home/owner" with current user's home [replace-with-current-user.py, which is automatically called by install, will change /home/owner to current user's directory in each script that install copies to $HOME/chunkymap])  
	then go to Terminal and run:  
	`minetestserver`  
	then when it is finished loading, press Ctrl C then run:  
    `chmod +x install-chunkymap-on-ubuntu.sh && ./install-chunkymap-on-ubuntu.sh`  
	
	Installing as cron job is OPTIONAL (and NOT recommended):
	* IF you are using a distro such as Ubuntu 14.04 where first line of /etc/crontab is "m h dom mon dow user command" then if you want regular refresh of map then run:
	(otherwise first edit the script to fit your crontab then)
	(if you are not using /var/www/html/minetest/chunkymapdata, edit chunkymap-cronjob script to use the correct directory, then)
    `chmod +x set-minutely-crontab-job.sh && ./set-minutely-crontab-job.sh`
* IF you are using Linux
	* Rename viewchunkymap.php so it won't be overwritten on update if you want to modify it (or anything you want) then make a link to it on your website or share the link some other way.
	# The commands below will work if you are using the web installer, or have done mv minetest-chunkymap-master "$HOME/Downloads/minetest-chunkymap" (and if you are using /var/www/html/minetest -- otherwise change that below)
	MT_MY_WEBSITE_PATH=/var/www/html/minetest
	sudo cp -f "$HOME/Downloads/minetest-chunkymap/web/chunkymap.php" "$MT_MY_WEBSITE_PATH/chunkymap.php"
	sudo cp -f "$HOME/Downloads/minetest-chunkymap/web/viewchunkymap.php" "$MT_MY_WEBSITE_PATH/viewchunkymap.php"
	sudo cp -R --no-clobber "$HOME/Downloads/minetest-chunkymap/web/images/*" "$MT_MY_WEBSITE_PATH/images/"
	#--no-clobber: do not overwrite existing
	# after you do this, the update script will do it for you if you are using /var/www/html/minetest, otherwise edit the update script before using it to get these things updated
* IF you are using Windows
	* Install Python 2.7
	* Run install-chunkymap-on-windows.bat
	(which just runs C:\Python27\python install-chunkymap-on-windows.py)
	(the installer will automatically download and install numpy and Pillow)
	* or OPTIONALLY manual install what the above script does:
		* put these files anywhere
		* python 2.7.x such as from python.org
		* run get_python_architecture.py to make sure you know whether to download the following in 32-bit or 64-bit  
		Administrator Command Prompt (to find it in Win 10, right-click windows menu)
		* update python package system:  
			`C:\python27\python -m pip install --upgrade pip wheel setuptools`
		* numpy such as can be installed via the easy unofficial installer wheel at  
		http://www.lfd.uci.edu/~gohlke/pythonlibs/#numpy  
		then:  
		cd to the folder where you downloaded the whl file  
			`C:\python27\python -m pip install "numpy-1.10.4+mkl-cp27-cp27m-win32.whl"`  
		(but put your specific downloaded whl file instead)  
		* Pillow (instead of PIL (Python Imaging Library) which is a pain on Windows): there is a PIL installer wheel for Python such as 2.7 here:  
		http://www.lfd.uci.edu/~gohlke/pythonlibs/  
		as suggested on http://stackoverflow.com/questions/2088304/installing-pil-python-imaging-library-in-win7-64-bits-python-2-6-4  
		then:  
			`C:\python27\python -m pip install "Pillow-3.1.1-cp27-none-win32.whl"`  
		(but put your specific downloaded whl file instead, such as Pillow-3.1.1-cp27-none-win_amd64.whl)
		* run (or if your python executable does not reside in C:\Python27\ then first edit the file):
		chunkymap-regen-loop.bat
		(all the batch does is run C:\Python27\python chunkymap-regen.py)
		(chunkymap-regen.py will ask for configuration options on first run and ask for your www root)

## Known Issues
* webapp: save selected world to a config file (click world on first visit to write initial config) instead of being silently autoselected
* Fix chunk generation and draw decachunks to canvas (so singleimage.py is not required to be run before chunkymap-regen.py)
* Detect exceptions in mintestmapper (such as database locked) and do NOT mark the chunk as is_empty
* Move the following to config dict:
    python_exe_path
* chunkymap.php should read the size of the chunks -- see near is_file($chunk_genresult_path) in chunkymap.php
* optionally hide player location
* Make a method (in chunkymap.php) to echo the map as an html5 canvas (refresh players every 10 seconds, check for new map chunks every minute)
