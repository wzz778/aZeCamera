# 前言
最近写了个招新面试系统，要求能支持拍照上传简历图片。经过对其义务，api的了解。用** React+TypeScript **手写出了一个原生的拍照上传组件，写此博客对此加以记录，也会公开到github方便日后的使用。
# 核心代码函数
## 1.调用摄像头
主要使用的是浏览器自带的api
```typescript
navigator.mediaDevices.getUserMedia
```
> 该api可以调用浏览器的摄像头权限，返回的是promise函数

```typescript
  mediaStream.getTracks().forEach((track) => {
    track.stop();
  });
```
具体代码：
```typescript
useEffect(() => {
  //组件挂载调用函数
  navigator.mediaDevices
    .getUserMedia({
      video: {
        width: 1080,
        height: 1920,
        frameRate: 30,
        facingMode: state.facing//调整摄像头的正反面
      },
      audio: false
    })
    .then((stream) => {
      mediaStream = stream;
      if (videoRef.current) {
        videoRef.current.srcObject = stream;
        videoRef.current.play();
      }
    })
    .catch((err) => {
      console.log(`An error occurred: ${err}`);
    });
  return () => {
    // 组件卸载时的逻辑
    if (mediaStream) {
      mediaStream.getTracks().forEach((track) => {
        track.stop();
      });
    }
  };
}, [state.facing]);
```
## 2.处理视频可播放事件
根据组件大小，计算视频流的宽高
```typescript
const handleCanPlay = () => {
  if (!state.streaming) {
    const height =
      (videoRef.current?.videoHeight as number) /
      (((videoRef.current?.videoWidth as number) / state.width) as number);
    setState({
      ...state,
      streaming: true,
      height: height as number
    });
  }
};
```
## 3.处理拍照事件
使用canvas来绘画视频流得出的照片结果
> 并调用onUploadPhoto回调函数将照片数据URL传递给父组件。

```typescript
const handleTakePhoto = () => {
  if (videoRef.current && canvasRef.current) {
    const { width, height } = state;
    canvasRef.current.width = width;
    canvasRef.current.height = height;
    const context = canvasRef.current.getContext('2d');
    if (context) {
      context.drawImage(videoRef.current, 0, 0, width, height);
      const photo = canvasRef.current.toDataURL('image/png');
      setState({ ...state, photo });
      onUploadPhoto(photo);
    }
  }
};
```
# 代码的封装
## 1.依赖引入
```typescript
import React, { useRef, useState, useEffect, ReactNode } from 'react';
```
## 2.定义组件接口类型：
```typescript
interface CameraProps {
  children?: ReactNode;
  onUploadPhoto: (photo: string) => void;//上传的函数
  onUnloadPhoto: (unload: boolean) => void;//卸载的函数
}
```
> CameraProps接口定义了组件的属性类型。它包含了可选的children属性，以及onUploadPhoto和onUnloadPhoto两个回调函数属性。

## 3.定义组件状态
```typescript
interface CameraState {
  streaming: boolean;
  width: number;
  height: number;
  photo: string | undefined;
  facing: 'user' | 'environment';
}
```
> CameraState接口定义了组件的状态类型。它包含了streaming表示是否正在录制、width和height表示视频的宽度和高度、photo表示拍摄的照片数据URL，以及facing表示相机的朝向。

## 4.定义相机组件：
```typescript
const Camera: React.FC<CameraProps> = ({ onUploadPhoto, onUnloadPhoto }) => {
  // ...
}
```
> Camera是一个函数组件，接受CameraProps作为属性。组件内部使用了useState和useRef来创建状态和引用。

## 5.定义组件的状态和引用：
```typescript
const videoRef = useRef<HTMLVideoElement>(null);
const canvasRef = useRef<HTMLCanvasElement>(null);
let mediaStream: MediaStream | null = null;
const [state, setState] = useState<CameraState>({
  streaming: false,
  width: 320,
  height: 0,
  photo: undefined,
  facing: 'environment'
});
```
> 这段代码使用useRef创建了videoRef和canvasRef两个引用，分别指向视频元素和画布元素。mediaStream是一个变量用于存储媒体流对象。useState创建了state状态对象和对应的setState函数。

## 6.处理切换相机事件：
```typescript
const handleToggleFacing = () => {
  setState({ ...state, facing: state.facing === 'user' ? 'environment' : 'user' });
};
```
## 7.渲染组件：
```typescript
return (
  <div>
    <video ref={videoRef} onCanPlay={handleCanPlay} />
    <canvas ref={canvasRef} style={{ display: 'none' }} />
    {state.streaming && (
      <div>
        <button onClick={handleTakePhoto}>Take Photo</button>
        <button onClick={handleToggleFacing}>Toggle Camera</button>
      </div>
    )}
    {state.photo && <img src={state.photo} alt="Captured" />}
    {children}
  </div>
);
```
# 完整代码：
## github
[https://github.com/wzz778/aZeCamera](https://github.com/wzz778/aZeCamera)
## 使用效果
手动马赛克🤣
### 拍照界面
![image.png](https://cdn.nlark.com/yuque/0/2023/png/26685644/1691053981755-729789b6-4230-4874-8fd8-8b8362b22008.png#averageHue=%23b0b0a6&clientId=u7c1774ef-c32d-4&from=paste&height=343&id=u262793db&originHeight=755&originWidth=465&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=311101&status=done&style=none&taskId=u7f54c9f3-6a7c-4d9f-af3f-43086680f3d&title=&width=211)
### 上传界面
![image.png](https://cdn.nlark.com/yuque/0/2023/png/26685644/1691053934026-4740154b-3a72-4bcc-ac59-cfe407aca8ed.png#averageHue=%23938b7d&clientId=u7c1774ef-c32d-4&from=paste&height=339&id=ue36aac12&originHeight=727&originWidth=463&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=327695&status=done&style=none&taskId=u548bbf19-b5c3-4f01-a4cf-bb5c2b94d5c&title=&width=215.66668701171875)
点击上传，调用handleUploadPhoto函数，返回图片的64编码
## 后言
功能pc，移动端都能使用，但样式主要适配了移动端，样式也可根据自己需求自行调整。
