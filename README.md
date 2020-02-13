# platform-sdk-js
SDK for online torrent streaming

## Features
1. Online torrent content streaming without waiting for full download
2. On-the-fly content transcoding
3. Downloading torrent as ZIP-archive
4. Subtitle transcoding, srt to vtt
5. OpenSubtitles support 
6. Streaming of external resources (including transcoding)

## Supported formats
* Video: avi, mkv, mp4, webm, m4v
* Audio: mp3, wav, ogg, flac, m4a
* Images: png, gif, jpg, jpeg
* Subtitles: srt, vtt

## Prerequisites
Before moving further we must be sure that webtor backend api is already installed.
Just follow [this guide](https://github.com/webtor-io/helm-charts) to setup all the things.

## Installation
```bash
npm install @webtor/platform-sdk-js
```
or
```bash
yarn add @webtor/platform-sdk-js
```

## Basic usage
Just a simple example of fetching torrent by magnet-uri, getting streaming url and stats:
```js
import webtor from '@webtor/platform-sdk-js';

const sdk = webtor({
  apiUrl: '...',
});

const magnetUri = '...';

const torrent = await sdk.magnet.fetchTorrent(magnetUri);

const expire = 60*60*24;

await sdk.torrent.push(torrent, expire);

const seeder = sdk.seeder.get(torrent.infoHash);

const filePath = torrent.files[0].path;

const url = await seeder.streamUrl(filePath);
console.log(url);

// NOTE: stats will become available only after content url access
seeder.stats(filePath, (path, data) => {
  console.log(data);
});
```
You can find fully working example [here](https://github.com/webtor-io/platform-sdk-js/tree/master/example)!

## Initialization

### `const sdk = webtor(options)`

`options` should be the following:

```
{
  apiUrl: String,              // API url (required)
  apiKey: String,              // API key
  downloadUrl: String,         // All download urls will contain this location
  tokenUrl: String,            // API will get access-token from this location
  tokenRenewInterval: Number,  // Renews access-token after specific period in ms
  grpcDebug: Boolean,          // If enabled shows grpc-web debug output
  statsRetryInterval: Number,  // If stats not available for this file it will retry after specific period in ms (default=3000)
  getToken: function() Promise // If defined custom token-function will be used 
}
```

## sdk.torrent
Provides functions for storing and retriving torrent-files.

### `sdk.torrent.fromUrl(url)`
Fetches parsed torrent from specific url. Be sure that appropriate CORS-headers are set on server-side in case of cross-site request (`Access-Control-Allow-Origin: *`). 

### `sdk.torrent.pull(infoHash)`
Pulls parsed torrent by infoHash from torrent-store.

### `sdk.torrent.push(torrent, expire)`
Pushes parsed torrent to torrent-store with specific expiration time in seconds. We can use any parsed torrent generated by [parse-torrent](https://github.com/webtorrent/parse-torrent) there.

### `sdk.torrent.touch(torrent, expire)`
Just renews expiration date of torrent-file.

## sdk.magnet
Provides functions for working with magnet-urls.

### `sdk.magnet.fetchTorrent(magnet)`
Fetches parsed torrent by magnet-uri.

## sdk.seeder
Provides functions for retriving and streaming torrent-content.

### `const seeder = sdk.seeder.get(infoHash)`
Returns seeder instance for the specific torrent stored in torrent-store.

### `seeder.url(path)`
Returns `url` for original source file.

### <a name="seeder-stream-url">`seeder.streamUrl(path, viewSettings = {})`</a>
Returns url capable for web-streaming.
`viewSettings` should be provided only in case of content trnascoding.
Possible `viewSettings`:
```
{
  a: Number,   // Selected audio channel
  s: Number,   // Selected subtitle channel
}
```

### <a name="seeder-stream-subtitle-url">`seeder.streamSubtitleUrl(path, viewSettings = {})`</a>
Returns subtitle-url capable for web-streaming. Can be used only for hls-transcoded content.
`viewSettings` should be provided only in case of content trnascoding.
Possible `viewSettings`:
```
{
  a: Number,   // Selected audio channel
  s: Number,   // Selected subtitle channel
}
```

### <a name="seeder-media-info">`seeder.mediaInfo(path)`</a>
Returns FFmpeg probing data. If content should not be transcoded only empty object `{}` will be returned.

### `seeder.downloadUrl(path)`
Returns download url.

### `seeder.zipUrl(path)`
Returns download url of zip-archive. Generates zip-archive of all files under directory and subdirectories specified by the `path`.

### <a name="seeder-opensubtitles">`seeder.openSubtitles(path)`</a>
Returns object that includes all subtitles from [OpenSubtitles.org](https://opensubtitles.org). Automatically generates streaming urls for every subtitle (srt => vtt encoding).

### `const statClient = seeder.stats(path, function(path, data) { ... })`
Receives status of specific file. Possible data:
```
{
  total: Number     // Number of total bytes
  completed: Number // Number of completed bytes
  peers: Number     // Total number of peers
  piecesList: []    // Array of pieces
}
```
Also returns instance of stat client.

### `statClient.close()`
Closes stat client

## sdk.ext
Provides functions for retriving and streaming external resources.

### `sdk.ext.url(url)`
Returns `url` for original external file proxied by the api. Can resolve CORS-issues.

### `sdk.ext.streamUrl(url, viewSettings = {})`
Same as [`seeder.streamUrl`](#seeder-stream-url) but for external resource.

### `sdk.ext.streamSubtitleUrl(url, viewSettings = {})`
Same as [`seeder.streamSubtitleUrl`](#seeder-stream-subtitle-url) but for external resource.

### `sdk.ext.mediaInfo(url)`
Same as [`seeder.mediaInfo`](#seeder-media-info) but for external resource.

### `sdk.ext.openSubtitles(url)`
Same as [`seeder.openSubtitles`](#seeder-opensubtitles) but for external resource.
