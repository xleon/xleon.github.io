---
layout:     post
title:      Large file downloads on Windows 10 mobile
date:       2017-04-07
summary:    Download safely in the background with progress feedback
categories: uwp windows download large-file xamarin
---
To download big files correctly (especially on mobile devices) we should try to achieve the following goals:

- Separate the download process from the current view in a background task: we don´t want the download to stop if we navigate to another view or the device goes idle
- The download stream __should not write directly to memory__: the device may run out of memory very soon, crashing your app. Instead, we should write to the file system directly as soon as the bytes are downloaded.
- Report progress: users should know how long it will take to finish. Otherwise it may look like it´s stuck

The same rules apply to any operating system, like iOS and Android (that will come in a future post).

It turns out that doing all this on Windows RT is fairly easy thanks to the class `Windows.Networking.BackgroundTransfer.BackgroundDownloader`, because it will follow all those rules:

{% highlight csharp %}
var downloader = new BackgroundDownloader();
var download = downloader.CreateDownload(uri, destinationFile);
download.Priority = BackgroundTransferPriority.High;

var progressCallback = new Progress<DownloadOperation>(operation =>
{
    if (download.Progress.TotalBytesToReceive > 0)
    {
        var percent = download.Progress.BytesReceived * 100 / download.Progress.TotalBytesToReceive;
        // TODO report percent
    }
});

await download.StartAsync().AsTask(progressCallback);
// at this point the download has been completed
{% endhighlight %}

That´s it actually, but in order to implement this in a cross-platform way (thinking about iOS and Android and Xamarin.Forms) I created and event `OnDownloadPercentProgress` and wrapped it all with an interface that is platform agnostic:

{% highlight csharp %}
using System;
using System.Threading.Tasks;
using PCLStorage;

namespace Core.Helpers
{
    public interface IFileManager
    {
        /// <summary>
        /// Parameter double is the download percent
        /// Parameter string is the uri string currently being downloaded
        /// </summary>
        event Action<double, string> OnDownloadPercentProgress;


        Task<IFile> Download(Uri uri, IFolder downloadFolder, string desiredFileName = null);
    }
}
{% endhighlight %}

`IFolder` comes from the cross-platform storage library called [PCLStorage](https://github.com/dsplaisted/PCLStorage)

And here is the implementation on Windows:

{% highlight csharp %}
using System;
using System.IO;
using System.IO.Compression;
using System.Threading.Tasks;
using Windows.Networking.BackgroundTransfer;
using Windows.Storage;
using Core.Helpers;
using PCLStorage;

namespace UWP.Helpers
{
    public class FileManager : IFileManager
    {
        public event Action<double, string> OnDownloadPercentProgress;

        public async Task<IFile> Download(Uri uri, IFolder downloadFolder, string desiredFileName = null)
        {
            desiredFileName = desiredFileName ?? Path.GetFileName(uri.LocalPath);

            var localFolder = await StorageFolder.GetFolderFromPathAsync(downloadFolder.Path);
            var destinationFile = await localFolder.CreateFileAsync(desiredFileName, Windows.Storage.CreationCollisionOption.GenerateUniqueName);

            var downloader = new BackgroundDownloader();
            var download = downloader.CreateDownload(uri, destinationFile);
            download.Priority = BackgroundTransferPriority.High;

            var progressCallback = new Progress<DownloadOperation>(operation =>
            {
                if (download.Progress.TotalBytesToReceive > 0)
                {
                    var percent = download.Progress.BytesReceived * 100 / download.Progress.TotalBytesToReceive;
                    OnDownloadPercentProgress?.Invoke(percent, uri.ToString());
                }
            });

            await download.StartAsync().AsTask(progressCallback);

            return await downloadFolder.GetFileAsync(desiredFileName);
        }
    }
}
{% endhighlight %}