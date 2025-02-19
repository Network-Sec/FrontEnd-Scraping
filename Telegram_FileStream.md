## Telegram Content Scraper
Primarily intended for file streams. 

Will include info, filesize, content (like password) and media img / video url. It will not download that files or media, you need to do it yourself. It only makes a list. 

```javascript
/* ** NOTE: You need to replace <groupID> with something like 123456 (real group ID you want to scrape),   */
/* ******** but leave <message-id> (literally) for dynamic replacement by the script.                      */
/* EXAMPLE: const customLink = "https://t.me/c/123456/<message-id>";                                       */
const customLink = "https://t.me/c/<groupID>/<message-id>";

const messageData3 = {};

function collectMessageData() {
    const messageGroups = document.getElementsByClassName('message-date-group');

    Array.from(messageGroups).forEach(group => {
        const bottomMarkers = group.querySelectorAll('.bottom-marker');

        bottomMarkers.forEach(marker => {
            const messageId = marker.getAttribute('data-message-id');

            if (messageId && !messageData3[messageId]) {
                const messageElement = marker.closest('div[id^="message-"]');

                const fileTitle = messageElement ? .querySelector('.file-info .file-title') ? .textContent.trim() || '';
                const fileSubTitle = messageElement ? .querySelector('.file-info .file-subtitle') ? .textContent.trim() || '';
                const textContent = messageElement ? .querySelector('.content-inner .text-content') ? .textContent.trim() || '';
                const mediaContent = messageElement ? .querySelector('.media-inner') ? .textContent.trim() || '';
                const mediaElements = messageElement ? .querySelectorAll('.media-inner img, .media-inner video');
                
                const mediaUrls = [];

                if (mediaElements) {
                    mediaElements.forEach(media => {
                        if (media.tagName.toLowerCase() === 'img') {
                            const src = media.src.trim();
                            if (src) mediaUrls.push(src);
                        } else if (media.tagName.toLowerCase() === 'video') {
                            const src = media.src || media.currentSrc;
                            if (src) mediaUrls.push(src.trim());
                        }
                    });
                }

                const link = customLink.replace('<message-id>', messageId);

                messageData3[messageId] = {
                    link: link,
                    fileTitle: fileTitle,
                    fileSize: fileSubTitle,
                    textContent: textContent,
                    mediaContentRaw: mediaContent,
                    mediaContent: mediaUrls
                };

                console.log(`Collected data for message ID: ${messageId}`, messageData3[messageId]);
            }
        });
    });
}

function startCollecting() {
    const intervalId = setInterval(collectMessageData, 500);
    return intervalId;
}

// Start the collection process
const collectionIntervalId = startCollecting();

// To stop the collection process, call:
// clearInterval(collectionIntervalId);
```

## Export it via BlobStorage
Just like creating forms for `CSRF` we can add and trigger a link to download stuff. 

```javascript
const dataStr = JSON.stringify(messageData3, null, 2);
const blob = new Blob([dataStr], { type: 'application/json' });
const url = URL.createObjectURL(blob);
const a = document.createElement('a');
a.href = url;
a.download = 'messageData.json'; 
document.body.appendChild(a); a.click(); 
document.body.removeChild(a); URL.revokeObjectURL(url); 
```
