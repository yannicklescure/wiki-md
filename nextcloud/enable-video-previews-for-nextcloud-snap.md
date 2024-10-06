# how-to-enable-video-previews-for-nextcloud

To enable video previews in your Nextcloud instance, you need to ensure that a few dependencies are installed, and then adjust your Nextcloud configuration to generate video thumbnails. Here’s how you can do it in the Snap version of Nextcloud.

## Steps to Enable Video Previews

### Install FFmpeg

The Nextcloud Snap package uses external tools like `ffmpeg` to generate video previews. To install `ffmpeg`, follow those steps:

- Create `/var/snap/nextcloud/bin/` directory

- Google for ffmpeg static build (has included static libraries). I used [https://johnvansickle.com/ffmpeg/](https://johnvansickle.com/ffmpeg/)

- Unpack ffmpeg and ffprobe into `/var/snap/nextcloud/bin/`

  ```bash
  tar -xvJf ffmpeg-release-amd64-static.tar.xz
  ```

### Edit `config.php`

You need to add the supported video file types to the `config.php` file in the Nextcloud configuration to enable preview generation for videos.

- First, open the `config.php` file:

  ```bash
  sudo nano /var/snap/nextcloud/current/nextcloud/config/config.php
  ```

- Add or modify the following lines inside the configuration file:

  ```php
  'enabledPreviewProviders' => array (
    0 => 'OC\Preview\Movie',
    1 => 'OC\Preview\MP4',
    2 => 'OC\Preview\AVI',
    3 => 'OC\Preview\PNG',
    4 => 'OC\Preview\JPEG',
    5 => 'OC\Preview\GIF',
    6 => 'OC\Preview\BMP',
    7 => 'OC\Preview\XBitmap',
    8 => 'OC\Preview\MP3',
    9 => 'OC\Preview\TXT',
    10 => 'OC\Preview\MarkDown',
    11 => 'OC\Preview\PDF',
  ),
  ```

- Update nexcloud config:

  ```bash
  sudo nextcloud.occ maintenance:mimetype:update-js
  sudo nextcloud.occ maintenance:mimetype:update-db
  ```

  This configuration enables video preview generation for MP4, AVI, and other file formats.

### Increase Memory Limit for Previews (Optional)

Generating video previews can be resource-intensive, especially for larger video files. You may want to increase the memory limit in the `config.php`:

```php
'preview_max_x' => 1024, // Adjust the width of the preview
'preview_max_y' => 768,  // Adjust the height of the preview
'preview_max_memory' => 4096, // Increase memory limit for preview generation
```

Furthermore, increase php memory limit configuration:

```bash
sudo snap set nextcloud php.memory-limit=4096M
```

### Resolving ffmpeg path

`ffmpeg` is now located at `/var/snap/nextcloud/bin/ffmpeg`. Yet Nextcloud isn't generating previews. The solution is to explicitly point it towards ffmpeg in config.php by setting:


```php
'preview_ffmpeg_path' => '/var/snap/nextcloud/bin/ffmpeg'
```

### Install the Preview Generator App

The Preview Generator app allows you to pre-generate thumbnails for media files, improving performance and user experience.

- Install the Preview Generator app:

  ```bash
  sudo nextcloud.occ app:install previewgenerator
  ```

- Enable the Preview Generator app:

  ```bash
  sudo nextcloud.occ app:enable previewgenerator
  ```

- After instalation generate all previews:

  ```bash
  sudo nextcloud.occ preview:generate-all
  ```

- As root, add a cron job (`crontab -e`) to generate new preview every 10 minutes:

  ```bash
  */10  * * * * nextcloud.occ preview:pre-generate
  ```

### Restart Nextcloud

Restart the Nextcloud Snap instance to apply the changes:

```bash
sudo snap restart nextcloud
```

After these steps, Nextcloud should generate video previews for supported file formats.

### Sources

- [https://help.nextcloud.com/t/preview-generation-mp4-ffmpeg/95937/11](https://help.nextcloud.com/t/preview-generation-mp4-ffmpeg/95937/11)
- [https://anto.online/how-to-enable-video-previews-for-nextcloud/](https://anto.online/how-to-enable-video-previews-for-nextcloud/)
- [https://help.nextcloud.com/t/thumbnail-preview-generation/34123/14](https://help.nextcloud.com/t/thumbnail-preview-generation/34123/14)
- [https://docs.nextcloud.com/server/stable/admin_manual/configuration_server/config_sample_php_parameters.html#preview-ffmpeg-path](https://docs.nextcloud.com/server/stable/admin_manual/configuration_server/config_sample_php_parameters.html#preview-ffmpeg-path)
- [https://github.com/nextcloud-snap/nextcloud-snap/issues/1046#issuecomment-1374427458](https://github.com/nextcloud-snap/nextcloud-snap/issues/1046#issuecomment-1374427458)
