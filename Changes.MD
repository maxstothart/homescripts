# Change Log

Here is the list of changes that I have to keep an idea of what has changed over time.

* 11/16/2021 - This is a pretty large update as I've moving away from Google Drive and migrating all my data to Dropbox.
* 6/3/2021 - Updated the rclone.service to reduce the pacer sleep time since I found my quotas are increased
* 6/1/2021 - I made a comment for the rclone.service for seeding libraries as I found 1M seemed
to make things slower for play startup overall so I left it back at the default for normal use
* 5/31/2021 - Updated --vfs-read-chunk-size to 1M to reduce download bandwidth on initial library scans. This increases API usage on certain things, but API limits are 1 billion per day so does not seem to be an issue.
* 9/4/2020 - With the new release of rclone, I have added in the vfs-cache-mode full backend for disk based caching and added in bandwidth per file controls as rclone is just too good at saturating my network
* 6/13/2020   - With the new changes to rclone, I've tested async and found no issues so I've removed that from mergerfs and rclone setup and let async run
* 6/5/2020    - Removed GD-Find and added a post exec action that runs async and returns immediately to warm up the directory cache
* 12/5/2019   - Removed attr-timeout since it caused some problems with nested files and caching
* 10/22/2019  - Initial change log to keep a running tally. Added attr-timeout 1000h to keep attributes cached longer.
