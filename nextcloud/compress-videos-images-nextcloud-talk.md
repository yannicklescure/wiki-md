# How to Automatically Compress Videos and Images from Nextcloud Talk

In today's digital age, communication platforms like Nextcloud Talk are essential for seamless collaboration. However, one challenge that often arises is managing the large media files exchanged during conversations—especially videos and images. These files can quickly consume server storage and slow down your Nextcloud instance.

To help streamline this process, I developed a simple yet effective Bash script for Ubuntu that automatically compresses videos and images shared on Nextcloud Talk. This script ensures that media files are optimized for storage and performance without sacrificing quality, helping you maintain a fast, efficient Nextcloud environment.

In this blog post, I’ll walk you through how the script works, the tools required for setup, and how you can customize it to suit your specific needs. Whether you're managing a personal Nextcloud server or handling large amounts of data for a team, this solution will make your media files more manageable and your Nextcloud Talk experience smoother. Let’s dive in!

## Pre-installation

```bash
sudo apt update
sudo apt install inotify-tools
```

## Script installation

```bash
sudo /usr/local/bin/nano resize_on_upload.sh
```

- Copy the following bash script:

```bash
#!/bin/bash

NEXTCLOUD_DATA_DIR="/var/snap/nextcloud/common/nextcloud/data"
TARGET_WIDTH=1080  # Desired width
LOG_FILE="/var/log/resize_files.log"

resize_image() {
    local FILE="$1"
    local USER_DIR="$2"

    while [[ ! -f "$FILE" ]]; do
        sleep 1
#        echo "Waiting for file."
    done

#    convert "$FILE" -resize "${TARGET_WIDTH}x>" -strip -interlace JPEG -quality 85 "$FILE"
    convert "$FILE" -resize "${TARGET_WIDTH}x>" "$FILE"
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $FILE resized" >> "$LOG_FILE"
    nextcloud.occ files:scan --path="${USER_DIR}"
}

resize_video() {
    local FILE="$1"
    local USER_DIR="$2"
    local TALK_DIR="$3"

    while [[ ! -f "$FILE" ]]; do
        sleep 1
#        echo "Waiting for file."
    done

    FILENAME=$(basename "$FILE")
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $FILENAME" >> "$LOG_FILE"
    BACKUP_FILE="$TALK_DIR/backup/${FILENAME%.*}_backup.mp4"

    mv "$FILE" "$BACKUP_FILE"

    echo "$(date '+%Y-%m-%d %H:%M:%S') - $BACKUP_FILE" >> "$LOG_FILE"

    ffmpeg -i "$BACKUP_FILE" -vf "scale=-2:720" -c:v libx264 -crf 28 -preset fast -c:a aac -b>
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $FILE resized" >> "$LOG_FILE"
    nextcloud.occ files:scan --path="${USER_DIR}"
}

monitor_directory() {
    local TALK_DIR="$1"
    local USER_DIR="$2"

    echo "$(date '+%Y-%m-%d %H:%M:%S') - Monitoring $TALK_DIR for new files..." >> "$LOG_FILE"

    inotifywait -m -r -e create --format '%w%f' "$TALK_DIR" | while read -r NEW_FILE; do
        # Wait until the file transfer is complete
        while [[ "$NEW_FILE" =~ \.(ocTransfer*|part)$ ]]; do
            sleep 1
            if [[ "$NEW_FILE" == *.ocTransfer* ]]; then
                NEW_FILE="${NEW_FILE%.ocTransfer*}"
            else
                NEW_FILE="${NEW_FILE%.part}"
            fi
        done

#        echo "$NEW_FILE"

        # Check if the new file is an image
        if [[ "$NEW_FILE" =~ \.(jpg|jpeg|png)$ ]]; then
            sleep 1
            echo "$(date '+%Y-%m-%d %H:%M:%S') - Resizing $NEW_FILE" >> "$LOG_FILE"
            resize_image "$NEW_FILE" "$USER_DIR"
        fi

        # Check if the new file is a video
        if [[ "$NEW_FILE" =~ \.(mp4)$ ]]; then
            sleep 1
            echo "$(date '+%Y-%m-%d %H:%M:%S') - Resizing $NEW_FILE" >> "$LOG_FILE"
            resize_video "$NEW_FILE" "$USER_DIR" "$TALK_DIR"
        fi
    done
}

# Loop through each user directory and start monitoring in the background
for user_dir in "$NEXTCLOUD_DATA_DIR"/*/; do
    TALK_DIR="${user_dir}files/Talk"
    USER_DIR="/$(basename $user_dir)/files/Talk"

    if [ -d "$TALK_DIR" ]; then
        monitor_directory "$TALK_DIR" "$USER_DIR" &
    else
        echo "$(date '+%Y-%m-%d %H:%M:%S') - Talk directory not found for user in $user_dir, >
    fi
done

# Wait for all background tasks to complete
wait
```

- Make the script executable:

```bash
sudo chmod +x resize_on_upload.sh
```

- Open cron as root

```bash
sudo crontab -e
```

- Add this line to the end of the file:

```bash
@reboot /usr/local/bin/resize_on_upload.sh &
```

- Reboot your server or start the script now:

```bash
sudo /usr/local/bin/./resize_on_upload.sh &
```
