To decrypt and stream a video in `react-native-video` version 4.4.5, you will need to handle the decryption process in a manner that allows you to pass the decrypted video data to the video player. Here is an approach you can take:

1. **Fetch Encrypted Video Data**: Use a method to fetch the encrypted video data from your server.
2. **Decrypt the Video Data**: Decrypt the fetched data using a suitable decryption algorithm.
3. **Stream Decrypted Video**: Feed the decrypted video data to the `react-native-video` component.

Here is an example demonstrating this process:

### 1. Fetch Encrypted Video Data
Use a method like `fetch` to get the encrypted video data.

```javascript
async function fetchEncryptedVideo(url) {
  const response = await fetch(url);
  const encryptedData = await response.arrayBuffer();
  return encryptedData;
}
```

### 2. Decrypt the Video Data
Use a decryption library that matches the encryption algorithm used. For example, if AES is used, you can use the `crypto-js` library.

First, install `crypto-js`:
```sh
npm install crypto-js
```

Then, decrypt the video data:

```javascript
import CryptoJS from 'crypto-js';

function decryptVideoData(encryptedData, key) {
  const decrypted = CryptoJS.AES.decrypt(
    CryptoJS.lib.CipherParams.create({ ciphertext: CryptoJS.enc.Base64.parse(encryptedData) }),
    CryptoJS.enc.Base64.parse(key)
  );
  return decrypted.toString(CryptoJS.enc.Base64);
}
```

### 3. Stream Decrypted Video
You need to convert the decrypted data into a Blob and use it in the video source:

```javascript
import React, { useState, useEffect } from 'react';
import { View, ActivityIndicator } from 'react-native';
import Video from 'react-native-video';

const VideoPlayer = ({ videoUrl, decryptionKey }) => {
  const [videoSource, setVideoSource] = useState(null);

  useEffect(() => {
    const loadVideo = async () => {
      const encryptedData = await fetchEncryptedVideo(videoUrl);
      const decryptedData = decryptVideoData(encryptedData, decryptionKey);
      const blob = new Blob([decryptedData], { type: 'video/mp4' });
      const url = URL.createObjectURL(blob);
      setVideoSource({ uri: url });
    };

    loadVideo();
  }, [videoUrl, decryptionKey]);

  if (!videoSource) {
    return <ActivityIndicator size="large" color="#0000ff" />;
  }

  return (
    <View style={{ flex: 1 }}>
      <Video
        source={videoSource}
        style={{ width: '100%', height: '100%' }}
        controls={true}
        resizeMode="contain"
      />
    </View>
  );
};

export default VideoPlayer;
```

### Important Notes:
1. **Performance**: Decrypting video data in real-time can be resource-intensive. Ensure that the decryption process is optimized to handle streaming efficiently.
2. **Security**: Be careful with how you manage and store decryption keys. Never hardcode them in your app.
3. **Compatibility**: The example assumes the video is in `mp4` format. Adjust the MIME type if you're dealing with a different video format.

This approach provides a basic framework for decrypting and streaming video in `react-native-video`. You may need to customize it further based on your specific requirements and the encryption method used.