# Additional Information

Extra information 
- [Non-Supported Characters](#Non-Supported-Characters)
- [Permission loss](#Permission-Loss)
- [Considerations](#Considerations)
- [Are Artifacts Zipped?](#Are-Artifacts-Zipped?)

## Non-Supported Characters

When uploading an artifact, the inputted `name` parameter along with the files specified in `files` cannot contain any of the following characters. They will be rejected by the server if attempted to be sent over and the upload will fail. These characters are not allowed due to limitations and restrictions with certain file systems such as NTFS. To maintain platform-agnostic behavior, all characters that are not supported by an individual filesystem/platform will not be supported on all filesystems/platforms.

- "
- :
- <
- \>
- |
- \*
- ?
- empty space

In addition to the aforementioned characters, the inputted `name` also cannot include the following
- \
- /


## Permission Loss

File permissions are not maintained between uploaded and downloaded artifacts. If file permissions are something that need to be maintained (such as an executable), consider archiving all of the files using something like `tar` and then uploading the single archive. After downloading the artifact, you can `un-tar` the individual file and permissions will be preserved.

```js
const artifact = require('@actions/artifact');
const artifactClient = artifact.create()
const artifactName = 'my-artifact';
const files = [
    '/home/user/files/plz-upload/my-archive.tgz',
]
const rootDirectory = '/home/user/files/plz-upload'
const uploadResult = await artifactClient.uploadArtifact(artifactName, files, rootDirectory)
```

## Considerations

During upload, each file is uploaded concurrently in 4MB chunks using a separate HTTPS connection per file. Chunked uploads are used so that in the event of a failure (which is entirely possible because the internet is not perfect), the upload can be retried. If there is an error, a retry will be attempted after a certain period of time.

Uploading will be generally be faster if there are fewer files that are larger in size vs if there are lots of smaller files. Depending on the types and quantities of files being uploaded, it might be beneficial to separately compress and archive everything into a single archive (using something like `tar` or `zip`) before starting and artifact upload to speed things up.

## Are Artifacts Zipped?

During upload, GZip is used to compress individual files before upload (if GZip does not reduce the file size, files are uploaded as-is). If a file gets uploaded with GZip, it is stored in the same format after being uploaded. GZip is used to significantly reduce the amount of data that is uploaded and to reduce the amount of HTTP calls made. This has huge performance implications particularly with self-hosted runners.

At the end of a run, you can download artifacts from the GitHub UI. To download all parts of an artifact at once, a zip file is created for all the individual files uploaded. Any previously GZipped files get decompressed and added to the Zip file. 

Long term there are plans for a more advanced UI for Artifacts that will allow artifact contents to be individually downloadable without any zipping.