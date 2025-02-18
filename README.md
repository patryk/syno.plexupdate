![GitHub release (latest by date)](https://img.shields.io/github/v/release/michealespinola/syno.plexupdate)
![GitHub top language](https://img.shields.io/github/languages/top/michealespinola/syno.plexupdate)
![GitHub](https://img.shields.io/github/license/michealespinola/syno.plexupdate)
[![Donate](https://img.shields.io/badge/Donate-PayPal-green.svg)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=2RYY4BETEQAJC)

 

![syno.plexupdate logo](./images/Syno.PLEX%20logo.png)

### Automatically Update Plex Media Server on the Synology NAS platform

# Description

This script takes into account many if not all of the issues I have previously read about for automatically updating Plex on the Synology NAS platform. This is a heavily-modified/overhauled version of the "[martinorob/plexupdate](https://github.com/martinorob/plexupdate)" script, with the specific intent to simplify its use to not require any Bash script variable editing or SSH access to the Synology NAS. This script originally started as a simple fork, but over the generations has turned into a wholly different script aside from the core task of updating Plex Media Server. The "fork" has been officially discontinued because it no longer resembles the original script, and has different support requirements.

Everything you need to do to get this script running can be done with basic DSM web administration by dropping this script onto the NAS and configuring a scheduled Task. This script is specifically for the update of the official Synology package of the Plex Media Server as released by [Plex GmbH](https://www.plex.tv/). This script utilizes Synology’s built-in tools to self-determine everything it needs to know about where Plex is located, how to update it, and to notify the system of updates or failures to the update process. If Plex is installed and properly configured, you will not have to edit this script for any details about the installation location of Plex. Public or Beta update channel selection follows what you have configured in the Plex Media Server's 'General' settings section.

Although only personally tested on my DS1019+, this script has been written to work on any compatible Synology platform using the built-in command-line utilities of the DSM. It reads your hardware architecture from the system and matches it against what is compatible with Plex. If its a part of the official Plex public or beta channel, this script will update it.

The default yet modifiable settings are that the script will not install an update unless it is 7+ days old. This is a stability safety-catch so that if a release has a bug, it is assumed it will be discovered and fixed within 7 days. It also keeps previously downloaded/installed packages in its "Updates" archive directory for 60 days before automatic deletion.

### DSM 6 and DSM 7 Support Notes

DSM 7 is officially supported starting from v4.0.0. DSM 6 support culminates with the v3.x.x series. Dual support may be considered in the future, but it is not at this time while the code is optimized and stablized for DSM 7, and after some new features are implimented.

* The latest updated version supporting DSM 6 can be found here:
  <https://github.com/michealespinola/syno.plexupdate/tree/master/Tools>

# How-To Setup Example

### 1. Save the Script to Your NAS

Download the script and place it into a location of your choosing. As an example, if you are using the "`admin`" account for system administration tasks, you can place the script within that accounts home folder; such as in a nested directory location like this:

    /home/scripts/bash/plex/syno.plexupdate/syno.plexupdate.sh

-or-

    /homes/admin/scripts/bash/plex/syno.plexupdate/syno.plexupdate.sh
    
**Note:** Synology recommends that you disable the default "`admin`" account for security reasons. In these examples, the admin directory structure is just a script storage location. You can run the script from here even if you disable the "`admin`" account.

### 2. Add the Plex 'Public Key Certificate' in DSM 6

* *This step is not required for DSM 7*

Updates directly from Plex (which is what this script installs) are not installable in the Synology DSM 6 by default - because no 3rd-party applications are. To install updates directly from Plex, DSM 6 must be configured to allow packages from other trusted application publishers. Plex's 'Public Key Certificate' must be installed to allow this safely without simply allowing any and all application publishers. The full instructions for this can be found on Plex's website here:

> <https://support.plex.tv/articles/205165858-how-to-add-plex-s-package-signing-public-key-to-synology-nas-package-center/>

1. Download the Plex 'Public Key Certificate' file from here: https://downloads.plex.tv/plex-keys/PlexSign.key
1. Open the [DSM](https://www.synology.com/en-global/knowledgebase/DSM/help) web interface
1. Open the [Package Center](https://www.synology.com/en-global/knowledgebase/DSM/help/DSM/PkgManApp/PackageCenter_desc)
   1. Click the General tab and change the Trust Level to "Synology Inc. and trusted publishers"
   1. Click on the Certificate tab and click the Import button
   1. Supply the location of the downloaded key file and import it
1. Click OK

### 3. Setup a Scheduled Task in the DSM

1. Open the [DSM](https://www.synology.com/en-global/knowledgebase/DSM/help) web interface
1. Open the [Control Panel](https://www.synology.com/en-global/knowledgebase/DSM/help/DSM/AdminCenter/ControlPanel_desc)
1. Open [Task Scheduler](https://www.synology.com/en-global/knowledgebase/DSM/help/DSM/AdminCenter/system_taskscheduler)
   1. Click Create -> Scheduled Task -> User-defined script
   1. Enter Task: name as '`Syno.Plex Update`', and leave User: set to '`root`'
   1. Click Schedule tab and configure per your requirements
   1. Click Task Settings tab
   1. Enter 'User-defined script' similar to:    
   '`bash /volume1/homes/admin/scripts/bash/plex/syno.plexupdate/syno.plexupdate.sh`'    
   ...if using the above script placement example. '`/volume1`' is the default storage volume on a Synology NAS. You can determine your script directory's full pathname by looking at the Location properties of the folder with the [File Station](https://www.synology.com/en-global/knowledgebase/DSM/help/FileStation/FileBrowser_desc) tool in the DSM:
      1. Right-click on the folder containing the script and choose Properties
      1. Copy the full directory path from the Location field
1. Click OK

# Script Logic Flow

1. Identify the "Plex Media Server" installation directory and other system-specific technical details
1. Create a "Packages" Archive directory if it does not exist, and remove old update package files
1. Extract Plex Token from local Preferences file for use to lookup available updates
1. Scrape JSON data to identify applicable updates specific to hardware architecture
1. Compare currently running version information against latest online version
1. If a new version exists and is older than the default 7-days - install the new version
1. Check if the upgrade was successful, update local changelog if applicable, and send appropriate notifications to the DSM and email (if configured for the DSM)
   * Notifications are only sent when an upgrade installation takes place

# Script Directory Structure

The script will automatically create directories and files as needed in the following directory structure based off of the location of the script, as per this example:

    /volume1/homes/admin/scripts/bash/plex/syno.plexupdate
    |   README.md
    |   syno.plexupdate.sh
    |
    \---Archive
       +---Packages
       |       changelog.txt
       |       PlexMediaServer-1.21.3.4021-5a0a3e4b2-x86_64.spk
       |       PlexMediaServer-1.21.3.4046-3c1c83ba4-x86_64.spk
       |       PlexMediaServer-1.21.4.4079-1b7748a7b-x86_64.spk
       |       PlexMediaServer-1.22.0.4163-d8c4875dd-x86_64.spk
       |       PlexMediaServer-1.22.1.4228-724c56e62-x86_64_DSM6.spk
       |       PlexMediaServer-1.22.2.4256-1e171f908-x86_64_DSM6.spk
       |       ...
       |
       \---Scripts
                syno.plexupdate.v2.3.1.sh
                syno.plexupdate.v2.3.2.sh
                syno.plexupdate.v2.3.3.sh
                syno.plexupdate.v2.9.9.sh
                syno.plexupdate.v2.9.9.1.sh
                syno.plexupdate.v2.9.9.2.sh
                ...

The Archive directory structure contains copies of update Packages as well as copies of the update Scripts that are running. The script archive is not a mirror of what is on GitHub. The script archive are a snapshot of running copies of the script. If you make modifications to the script, the copy in the script archive will be updated accordingly. The packages archive is similarly for manual rollback purposes.

The '`changelog.txt`' file is a historical changelog only for updates installed locally by the script. It will not contain any historical information for versions that were otherwised skipped by the script.

# Example Output

    Dear user,

    Task Scheduler has completed a scheduled task.

    Task: Syno.Plex Update
    Start time: Thu, 25 Mar 2021 02:05:36 GMT
    Stop time: Thu, 25 Mar 2021 02:06:21 GMT
    Current status: 1 (Interrupted)
    Standard output/error:

    SYNO.PLEX UPDATER SCRIPT v3.0.0

             Script: syno.plexupdate.sh v3.0.0
         Script Dir: /volume1/homes/admin/scripts/bash/plex/syno.plexupdate
        Running Ver: 3.0.0
         Online Ver: 2.3.3
           Released: 2020-09-06 06:34:14-07:00 (200+ days old)
                     * No new version found.

           Synology: DS1019+ (x86_64), DSM 6.2.4-25556 Update 0
           Plex Dir: /volume1/Plex/Library/Application Support/Plex Media Server
         Plex Token: ####################
        Running Ver: 1.22.0.4163-d8c4875dd
         Online Ver: 1.22.1.4228-724c56e62 (Public Channel)
           Released: 2021-03-23 05:41:44-07:00 (1+ days old)
                     * Newer version found!

        New Package: PlexMediaServer-1.22.1.4228-724c56e62-x86_64_DSM6.spk
        Package Age: 1+ days old (0+ required for install)

    INSTALLING NEW PACKAGE:
    ----------------------------------------
    2021-03-25 02:05:40 URL:https://downloads.plex.tv/plex-media-server-new/1.22.1.4228-724c56e62/synology/PlexMediaServer-1.22.1.4228-724c56e62-x86_64_DSM6.spk [116664320/116664320] -> "/volume1/homes/admin/scripts/bash/plex/syno.plexupdate/Archive/Packages/PlexMediaServer-1.22.1.4228-724c56e62-x86_64_DSM6.spk" [1]
    package Plex Media Server stop successfully

    /volume1/homes/admin/scripts/bash/plex/syno.plexupdate/Archive/Packages/PlexMediaServer-1.22.1.4228-724c56e62-x86_64_DSM6.spk install successfully

    package Plex Media Server start successfully
    ----------------------------------------

        Update from: 1.22.0.4163-d8c4875dd
                 to: 1.22.1.4228-724c56e62 succeeded!

    NEW FEATURES:
    ----------------------------------------
    * (Library) Improved search handling of non-Latin scripts (#7896)
    * (Library) Improved search handling of punctuation (#7833)
    * (Web) Updated to 4.52.2
    * (Web) Updated to 4.53.0
    * Updated Translations.
    ----------------------------------------

    FIXED FEATURES:
    ----------------------------------------
    * (Butler) The scheduled job to refresh local metadata could use a lot or memory.
    * (Gaming) Allow h/w encoding for parallel N64 core.
    * (Gaming) Fix for games not saving state on Windows.
    * (Gaming) Move to using nearest neighbor scaling for sharper rendering.
    * (Gaming) OpenGL v2 and v3 rendering pipelines.
    * (Gaming) Use passed in display dimensions to render at a higher resolution.
    * (Library) Artist genres were not being set for FLAC files.
    * (Library) Don't expose genre radio if user has "none" selected for genre source.
    * (Library) Ensure shuffle doesn't bias towards single-track artists.
    * (Library) Old lyrics weren't being removed, which could result in errors displaying lyrics.
    * (Library) TV intro detection would run unconditionally for users without a Plex Pass using the beta TV agent.
    * (Metadata) On rare occasions, refreshing a movie item may remove reviews and extras (#12428)
    * (Statistics) Don't store duplicate records in the statistics tables
    * (Transcoder) Short backwards seeks could fail in DASH transcodes under some circumstances (#11824)
    * Added Slovak and Slovenian translations.
    * Crash returning active playback sessions.
    * Episodes with air date and no episode number could play out of order.
    * The server could fail to start up on some ARM systems (#12513)
    * Tightened cross-origin request security restrictions (#7712)
    ----------------------------------------



    From SYNOLOGY

# Default Settings (config.ini)

The script utilizes (4) default but user-configurable variables:

1. `MinimumAge=7`
   * A **7**-day age requirement for installing the latest version as a bug/issue deterrent
1. `OldUpdates=60`
   * A **60**-day age requirement for deleting old package installer files
1. `NetTimeout=900`
   * A **900**-second (15 minute) network timeout for hung network connections
1. `SelfUpdate=0`
   * A 0=off 1=on toggle to enable self-updating to the latest packaged release. Change this to **1** to enable self-updating that follows the same minimum age requirement as the Plex updates

# Known Non-Issues

* If the script runs successfully, the DSM Task status will show "`Interrupted (1)`" and the notification email will state "`Current status: 1 (Interrupted)`". This exit/error status of (1) is intentionally caused by the script in order to force the DSM to perform an email notification of a successful update (in the form of an interruption/error). The DSM otherwise would only send notifications of failed task events, and notifications of successful Plex updates would not be possible.
* If DSM 6 is not configured to allow 3rd-party "trusted publishers", the script will log "`error = [289]`" during the package installation process. Synology DSM 6 has been known to sporadically "lose" 3rd-party security certificates for unknown reasons. If this happens, you will have to re-add the Plex 'Public Key Certificate' to your system. DSM 7 no longer has this requirement.

# Common Mistakes

* Bash shell scripts running on Linux must be saved as LF (Line Feed) sequenced text. Alternatively, Windows typically uses CRLF (Carriage Return Line Feed). If you inadvertantly save your script with CRLF instead of LF sequencing, it will error with verbiage such as:
  * `'\r': command not found`
  * `syntax error near unexpected`
  
  To resolve this issue you will need to [re]save your copy of the script with LF sequencing, or download a new copy directly from GitHub.

# Thank You's

Many thanks to:

1. [j0nsplex](https://forums.plex.tv/u/j0nsplex) for starting it all
1. [martinorob](https://github.com/martinorob) for giving me a jumping-off point
1. [ChuckPa](https://forums.plex.tv/u/ChuckPa) and [trumpy81](https://forums.plex.tv/u/trumpy81) at Plex for being kind and encouraging to this endeavour
1. [Kynch](https://forums.plex.tv/u/Kynch) for the idea of self-updating the script
1. [SunMar](https://forums.plex.tv/u/SunMar) for the idea of a changelog

 

[![paypal](https://www.paypalobjects.com/en_US/i/btn/btn_donateCC_LG.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=2RYY4BETEQAJC)
