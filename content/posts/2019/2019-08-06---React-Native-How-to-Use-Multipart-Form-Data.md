---
title: React Native - Sending an Image to an API to upload a file
date: "2019-08-06T19:27:37.121Z"
template: "post"
draft: false
slug: "/posts/React-Native-Multipart-Form-Data-to-API"
category: "React"
tags:
  - "React"
description: "How to send an image file to an API to have it uploaded to AWS S3"
---

The purpose of this code was to send an image file to an API in order to upload an image to `AWS S3` to retrieve an image URL.

In order to send a file to a server, you have to send it as a `multipart/form-data`. Since I am using an image, I used a library called `react-native-image-picker`. It is very easy to use the library--you just have to follow the instruction written in the official Github.

`npm i --save react-native-image-picker`

and then you just add the code given in the instruction .

```typescript
_addPicture = () => {
  ImagePicker.showImagePicker(options, (response) => {
    if (response.didCancel) {
      console.log("User cancelled image picker");
    } else if (response.error) {
      console.log("ImagePicker Error: ", response.error);
    } else if (response.customButton) {
      console.log("User tapped custom button: ", response.customButton);
    } else {
      const source = { uri: response.uri };

      this.setState({
        image: source,
        imageName: response.fileName,
      });
    }
  });
};
```

Now that you have added the picture, you have to create a `formData` in order to send the picture to an API.

```typescript
_createFormData = () => {
  const { image, imageName } = this.state;
  const imageUri = image.uri.slice(7);

  let picture = new FormData();
  let file = {
    uri: imageUri,
    type: "image/jpeg",
    name: imageName,
  };
  picture.append("image", file);
  return picture;
};
```

`image` is an object which contains a key called uri whose value contains some prefix that `multipart/form-data` cannot recognize. So, I used the `slice()` method in order to cut the first seven letters off the string.

```typescript
_uploadPicture = async () => {
  const accessToken = await AsyncStorage.getItem("@storage_Key");
  const picture = this._createFormData();
  const pictureUploadSettings = {
    method: "POST",
    headers: {
      Authorization: `Token ${accessToken}`,
      "Content-Type": "multipart/form-data",
    },
    body: picture,
  };

  const response = await fetch(
    `${API_URL}listener/upload/picture`,
    pictureUploadSettings
  );
  const pictureUri = await response.json();

  return pictureUri.picture_uri;
};
```

Then I sent the image file to the API using the code above.
