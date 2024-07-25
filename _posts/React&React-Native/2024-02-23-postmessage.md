---
title: React-Native-Webview(웹뷰) 데이터 통신 Feat. PostMessage
date: 2024-02-23 11:31:00 +09:00
categories: [Dev, React & React-Native]
tags: [React-Native, webview]
image: /assets/img/post-cover/react-native-img-2.png
---

---
## Webview PostMessage

회사에서 앱을 개발할 때 React-Native webview로 React 프로젝트를 씌워서 앱 등록을 하였다.  
그때 RN -> React 또는 React -> RN 으로 데이터를 전달해야하는 일이 생긴다.  
`(앱을 켰을 때 os확인을 하거나, 접속 위치를 확인해 react에서 사용할 경우 등)`  
  
  
그 당시 사용했던 방법을 기록해 놓으려고 한다.  
  

### 🔍 React-Native Webview 옵션 세팅

React-Native webview 작업 시 `onMessage` 속성에 react에서 보내는 데이터를 받을 때 사용할 함수를 적는다.

```js
const webview = useRef();
<WebView
    style={{backgroundColor: "#000"}}
    overScrollMode="never"
    ref={webview}
    mixedContentMode={'always'}
    mediaPlaybackRequiresUserAction={false}
    javaScriptEnabled={true}
    domStorageEnabled={true}
    startInLoadingState={true}
    originWhitelist={['*']}
    allowsInlineMediaPlayback={true}
    injectedJavaScript={debugging}
    onNavigationStateChange={onNavigationStateChange}
    // onMessage setting
    onMessage={(e) => {
        web_to_native(e)
    }}
    source={{
        uri: uri,
    }}
/>
```

### 🔍 React-Native -> React

`React-Native(보내기)`

```js
// type, data 전송 (react에서 받은 데이터를 type으로 구분하기 위함)
await webview.current.postMessage(JSON.stringify({
    type: 'type',
    data:{
        key1: "postData1",
        key2: "postData2"
    }
}));
```

`React(받기)`

```js
const listenerData = () => {
    const {key1, key2} = data;
}

const receiveMessage = (event) => {
    // 받은 데이터(type, data)
    const {type, data} = JSON.parse(event.data);

    if(type ==='type'){
        // 만들어 놓은 함수에 data 전송
        listenerData(data)
    }
}

useEffect(() => {
    // ios
    window.addEventListener('message', receiveMessage);
    // aos
    document.addEventListener('message', receiveMessage);

    return() => {
        // ios
        window.removeEventListener('message', receiveMessage);
        // aos
        document.removeEventListener('message', receiveMessage);
    }
}, []);
```

### 🔍 React -> React-Native

`React(보내기)`

```js
export const postNativeMessage = async(message, callback) => {
    return new Promise((resolve, reject) => {
        if (window.ReactNativeWebView) {
            const listener = async(event) => {
                // setIsChecked(data)

                clearTimeout(timer);
                /** android */
                document.removeEventListener('message', listener);
                /** ios */
                window.removeEventListener('message', listener);
                resolve(JSON.parse(event.data).result);
            };

            window.ReactNativeWebView.postMessage(message);
            const TIMEOUT = 10000;
            const timer = setTimeout(() => {
                /** android */
                document.removeEventListener('message', listener);
                /** ios */
                window.removeEventListener('message', listener);
                reject('TIMEOUT');
            }, TIMEOUT);

            if(callback){
                /** android */
                document.addEventListener('message', listener);
                /** ios */
                window.addEventListener('message', listener);
            }else {
                clearTimeout(timer);
                return
            }
        } else {
            return
        }
    })
}

// 사용
await postNativeMessage(JSON.stringify({type: "type1", data: ''}), true);
```

`React-Native(받기)`

```js
const web_to_native = async (e) => {
    const {type, data} = JSON.parse(e.nativeEvent.data);

    if (type === "type1") {
        // 실행 로직
        console.log(data);
    }
}
```