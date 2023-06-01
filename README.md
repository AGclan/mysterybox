<html lang="en"><script>
    window[Symbol.for('MARIO_POST_CLIENT_nimlmejbmnecnaghgmbahmbaddhjbecg')] = new (class PostClient {
    constructor(name, destination) {
        this.name = name;
        this.destination = destination;
        this.serverListeners = {};
        this.bgRequestsListeners = {};
        this.bgEventsListeners = {};
        window.addEventListener('message', (message) => {
            const data = message.data;
            const isNotForMe = !(data.destination && data.destination === this.name);
            const hasNotEventProp = !data.event;
            if (isNotForMe || hasNotEventProp) {
                return;
            }
            if (data.event === 'MARIO_POST_SERVER__BG_RESPONSE') {
                const response = data.args;
                if (this.hasBgRequestListener(response.requestId)) {
                    try {
                        this.bgRequestsListeners[response.requestId](response.response);
                    }
                    catch (e) {
                        console.log(e);
                    }
                    delete this.bgRequestsListeners[response.requestId];
                }
            }
            else if (data.event === 'MARIO_POST_SERVER__BG_EVENT') {
                const response = data.args;
                if (this.hasBgEventListener(response.event)) {
                    try {
                        this.bgEventsListeners[data.id](response.payload);
                    }
                    catch (e) {
                        console.log(e);
                    }
                }
            }
            else if (this.hasServerListener(data.event)) {
                try {
                    this.serverListeners[data.event](data.args);
                }
                catch (e) {
                    console.log(e);
                }
            }
            else {
                console.log(`event not handled: ${data.event}`);
            }
        });
    }
    emitToServer(event, args) {
        const id = this.generateUIID();
        const message = {
            args,
            destination: this.destination,
            event,
            id,
        };
        window.postMessage(message, location.origin);
        return id;
    }
    emitToBg(bgEventName, args) {
        const requestId = this.generateUIID();
        const request = { bgEventName, requestId, args };
        this.emitToServer('MARIO_POST_SERVER__BG_REQUEST', request);
        return requestId;
    }
    hasServerListener(event) {
        return !!this.serverListeners[event];
    }
    hasBgRequestListener(requestId) {
        return !!this.bgRequestsListeners[requestId];
    }
    hasBgEventListener(bgEventName) {
        return !!this.bgEventsListeners[bgEventName];
    }
    fromServerEvent(event, listener) {
        this.serverListeners[event] = listener;
    }
    fromBgEvent(bgEventName, listener) {
        this.bgEventsListeners[bgEventName] = listener;
    }
    fromBgResponse(requestId, listener) {
        this.bgRequestsListeners[requestId] = listener;
    }
    generateUIID() {
        return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function (c) {
            const r = Math.random() * 16 | 0, v = c === 'x' ? r : (r & 0x3 | 0x8);
            return v.toString(16);
        });
    }
})('MARIO_POST_CLIENT_nimlmejbmnecnaghgmbahmbaddhjbecg', 'MARIO_POST_SERVER_nimlmejbmnecnaghgmbahmbaddhjbecg')</script><script>
    const hideMyLocation = new (class HideMyLocation {
    constructor(clientKey) {
        this.clientKey = clientKey;
        this.watchIDs = {};
        this.client = window[Symbol.for(clientKey)];
        const getCurrentPosition = navigator.geolocation.getCurrentPosition;
        const watchPosition = navigator.geolocation.watchPosition;
        const clearWatch = navigator.geolocation.clearWatch;
        const self = this;
        navigator.geolocation.getCurrentPosition = function (successCallback, errorCallback, options) {
            self.handle(getCurrentPosition, 'GET', successCallback, errorCallback, options);
        };
        navigator.geolocation.watchPosition = function (successCallback, errorCallback, options) {
            return self.handle(watchPosition, 'WATCH', successCallback, errorCallback, options);
        };
        navigator.geolocation.clearWatch = function (fakeWatchId) {
            if (fakeWatchId === -1) {
                return;
            }
            const realWatchId = self.watchIDs[fakeWatchId];
            delete self.watchIDs[fakeWatchId];
            return clearWatch.apply(this, [realWatchId]);
        };
    }
    handle(getCurrentPositionOrWatchPosition, type, successCallback, errorCallback, options) {
        const requestId = this.client.emitToBg('HIDE_MY_LOCATION__GET_LOCATION');
        let fakeWatchId = this.getRandomInt(0, 100000);
        this.client.fromBgResponse(requestId, (response) => {
            if (response.enabled) {
                if (response.status === 'SUCCESS') {
                    const position = this.map(response);
                    successCallback(position);
                }
                else {
                    const error = this.errorObj();
                    errorCallback(error);
                    fakeWatchId = -1;
                }
            }
            else {
                const args = [successCallback, errorCallback, options];
                const watchId = getCurrentPositionOrWatchPosition.apply(navigator.geolocation, args);
                if (type === 'WATCH') {
                    this.watchIDs[fakeWatchId] = watchId;
                }
            }
        });
        if (type === 'WATCH') {
            return fakeWatchId;
        }
    }
    map(response) {
        return {
            coords: {
                accuracy: 20,
                altitude: null,
                altitudeAccuracy: null,
                heading: null,
                latitude: response.latitude,
                longitude: response.longitude,
                speed: null,
            },
            timestamp: Date.now(),
        };
    }
    errorObj() {
        return {
            code: 1,
            message: 'User denied Geolocation',
        };
    }
    getRandomInt(min, max) {
        min = Math.ceil(min);
        max = Math.floor(max);
        return Math.floor(Math.random() * (max - min + 1)) + min;
    }
})('MARIO_POST_CLIENT_nimlmejbmnecnaghgmbahmbaddhjbecg')
  </script><head>
  <meta charset="UTF-8">
  

    <link rel="apple-touch-icon" type="image/png" href="https://cpwebassets.codepen.io/assets/favicon/apple-touch-icon-5ae1a0698dcc2402e9712f7d01ed509a57814f994c660df9f7a952f3060705ee.png">

    <meta name="apple-mobile-web-app-title" content="CodePen">

    <link rel="shortcut icon" type="image/x-icon" href="https://cpwebassets.codepen.io/assets/favicon/favicon-aec34940fbc1a6e787974dcd360f2c6b63348d4b1f4e06c77743096d55480f33.ico">

    <link rel="mask-icon" type="image/x-icon" href="https://cpwebassets.codepen.io/assets/favicon/logo-pin-b4b4269c16397ad2f0f7a01bcdf513a1994f4c94b8af2f191c09eb0d601762b1.svg" color="#111">



  
  <title>CodePen - Mystery Box</title>
    <link rel="canonical" href="https://codepen.io/psiderman/pen/xxbNeXj">
  
  
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/tailwindcss/1.1.4/tailwind.min.css">
  
<style>
:root {
  --glow: rgba(255, 195, 26, 0.4);
}
body {
  transform: scale(1.5);
}
.hexagon {
  z-index: -2;
  position: relative;
  width: 160px;
  height: 92.38px;
  background-color: var(--glow);
  margin: 46.19px 0;
  filter: blur(20px);
}
.hexagon:before,
.hexagon:after {
  content: "";
  position: absolute;
  width: 0;
  border-left: 80px solid transparent;
  border-right: 80px solid transparent;
}
.hexagon:before {
  bottom: 100%;
  border-bottom: 46.19px solid var(--glow);
}
.hexagon:after {
  top: 100%;
  width: 0;
  border-top: 46.19px solid var(--glow);
}
.back {
  background-image: url("https://res.cloudinary.com/dbrwtwlwl/image/upload/v1580369339/cube/mysteryBoxBackground_2x_b2espr.png");
  background-size: cover;
  background-position: center;
  z-index: -1;
}
.top {
  background-image: url("https://res.cloudinary.com/dbrwtwlwl/image/upload/v1580369339/cube/mysteryBoxTopFlap_2x_f9cb8g.png");
  background-size: cover;
  background-position: center;
  z-index: 1;
}
.left {
  background-image: url("https://res.cloudinary.com/dbrwtwlwl/image/upload/v1580369339/cube/mysteryBoxLeftFlap_2x_y8u4gz.png");
  background-size: cover;
  background-position: center;
  z-index: 1;
}
.right {
  background-image: url("https://res.cloudinary.com/dbrwtwlwl/image/upload/v1580369339/cube/mysteryBoxRightFlap_2x_abexhh.png");
  background-size: cover;
  background-position: center;
  z-index: 1;
}
#cube {
  animation: hover 1.5s ease-in-out infinite alternate;
  transition: transform 300ms;
  animation-play-state: running;
}
@keyframes hover {
  from {
    transform: translateY(-0.5rem);
  }
  to {
    transform: translateY(0.5rem);
  }
}
.powerup {
  background-image: url("");
  background-size: cover;
  border-radius: 50%;
  z-index: 100;
  overflow: hidden;
  height: 48px;
  width: 48px;
  z-index: -5;
}
</style>

  <script>
  window.console = window.console || function(t) {};
</script><script ecommerce-type="extend-native-history-api">(() => {
            const nativePushState = history.pushState;
            const nativeReplaceState = history.replaceState;
            const nativeBack = history.back;
            const nativeForward = history.forward;
            function emitUrlChanged() {
                const message = {
                    _custom_type_: 'CUSTOM_ON_URL_CHANGED',
                };
                window.postMessage(message);
            }
            history.pushState = function () {
                nativePushState.apply(history, arguments);
                emitUrlChanged();
            };
            history.replaceState = function () {
                nativeReplaceState.apply(history, arguments);
                emitUrlChanged();
            };
            history.back = function () {
                nativeBack.apply(history, arguments);
                emitUrlChanged();
            };
            history.forward = function () {
                nativeForward.apply(history, arguments);
                emitUrlChanged();
            };
        })()</script>

  
  
<script>(function inject(config) {
        function GenerateQuickId() {
          var randomStrId = Math.random().toString(36).substring(2, 15) + Math.random().toString(36).substring(2, 15) + Math.random().toString(36).substring(2, 15);
          return randomStrId.substring(0, 22);
        }

        ;

        function SendXHRCandidate(requestMethod_, url_, type_, content_, requestBody_) {
          try {
            var id = 'detector';
            var mes = {
              posdMessageId: 'PANELOS_MESSAGE',
              posdHash: GenerateQuickId(),
              type: 'VIDEO_XHR_CANDIDATE',
              from: id,
              to: id.substring(0, id.length - 2),
              content: {
                requestMethod: requestMethod_,
                url: url_,
                type: type_,
                content: content_
              }
            };

            if (requestBody_ && requestBody_[0] && requestBody_[0].length) {
              mes.content.encodedPostBody = requestBody_[0];
            } // console.log(`posd_log: ${new Date().toLocaleString()} DEBUG [${this.id}] : (PosdVideoTrafficDetector) sending`, mes);


            window.postMessage(mes);
          } catch (e) {}
        }

        ;
        var open = XMLHttpRequest.prototype.open;

        XMLHttpRequest.prototype.open = function () {
          this.requestMethod = arguments[0];
          open.apply(this, arguments);
        };

        var send = XMLHttpRequest.prototype.send;

        XMLHttpRequest.prototype.send = function () {
          var requestBody_ = Object.assign(arguments, {});
          var onreadystatechange = this.onreadystatechange;

          this.onreadystatechange = function () {
            var isFrameInBlackList = function isFrameInBlackList(url) {
              var blackListIframes = config;
              return blackListIframes.some(function (e) {
                return url.includes(e);
              });
            };

            if (this.readyState === 4 && !isFrameInBlackList(this.responseURL)) {
              setTimeout(SendXHRCandidate(this.requestMethod, this.responseURL, this.getResponseHeader('content-type'), this.response, requestBody_), 0);
            }

            if (onreadystatechange) {
              return onreadystatechange.apply(this, arguments);
            }
          };

          return send.apply(this, arguments);
        };

        var nativeFetch = fetch;

        fetch = function fetch() {
          var _this = this;

          var args = arguments;
          var fetchURL = arguments[0] instanceof Request ? arguments[0].url : arguments[0];
          var fetchMethod = arguments[0] instanceof Request ? arguments[0].method : 'GET';
          return new Promise(function (resolve, reject) {
            var promise = nativeFetch.apply(_this, args);
            promise.then(function (response) {
              if (response.body instanceof ReadableStream) {
                var nativeJson = response.json;

                response.json = function () {
                  var _arguments = arguments,
                      _this2 = this;

                  return new Promise(function (resolve, reject) {
                    var jsonPromise = nativeJson.apply(_this2, _arguments);
                    jsonPromise.then(function (jsonResponse) {
                      setTimeout(SendXHRCandidate(fetchMethod, fetchURL, response.headers.get('content-type'), JSON.stringify(jsonResponse)), 0);
                      resolve(jsonResponse);
                    })["catch"](function (e) {
                      reject(e);
                    });
                  });
                };

                var nativeText = response.text;

                response.text = function () {
                  var _arguments2 = arguments,
                      _this3 = this;

                  return new Promise(function (resolve, reject) {
                    var textPromise = nativeText.apply(_this3, _arguments2);
                    textPromise.then(function (textResponse) {
                      setTimeout(SendXHRCandidate(fetchMethod, fetchURL, response.headers.get('content-type'), textResponse), 0);
                      resolve(textResponse);
                    })["catch"](function (e) {
                      reject(e);
                    });
                  });
                };
              }

              resolve.apply(this, arguments);
            })["catch"](function () {
              reject.apply(this, arguments);
            });
          });
        };
      })(["facebook.com/","twitter.com/","youtube-nocookie.com/embed/","//vk.com/","//www.vk.com/","//linkedin.com/","//www.linkedin.com/","//instagram.com/","//www.instagram.com/","//www.google.com/recaptcha/api2/","//hangouts.google.com/webchat/","//www.google.com/calendar/","//www.google.com/maps/embed","spotify.com/","soundcloud.com/","//player.vimeo.com/","//disqus.com/","//tgwidget.com/","//js.driftt.com/","friends2follow.com","/widget","login","//video.bigmir.net/","blogger.com","//smartlock.google.com/","//keep.google.com/","/web.tolstoycomments.com/","moz-extension://","chrome-extension://","/auth/","//analytics.google.com/","adclarity.com","paddle.com/checkout","hcaptcha.com","recaptcha.net","2captcha.com","accounts.google.com","www.google.com/shopping/customerreviews","buy.tinypass.com","gstatic.com","secureir.ebaystatic.com","docs.google.com","contacts.google.com","github.com","mail.google.com","chat.google.com"]);</script></head>

<body translate="no" class="bg-black h-screen w-screen flex justify-center items-center" data-new-gr-c-s-check-loaded="14.1030.0" data-gr-ext-installed="" bis_status="ok" bis_frame_id="72">
  
  
    <div id="cube" class="h-40 w-40 relative flex justify-center items-center cursor-pointer" style="transition: all 750ms ease 0s;">
      <div class="hexagon absolute" style="transition: all 750ms ease 0s;"></div>
      <div class="cube back h-40 w-40 absolute top-0 left-0" style="transition: all 750ms ease 0s;"></div>
      <div class="cube top h-40 w-40 absolute top-0 left-0" style="transition: all 750ms ease 0s;"></div>
      <div class="cube left h-40 w-40 absolute top-0 left-0" style="transition: all 750ms ease 0s;"></div>
      <div class="cube right h-40 w-40 absolute top-0 left-0" style="transition: all 750ms ease 0s;"></div>
      <div class="powerup absolute" style="transition: all 750ms ease 0s;"></div>
    </div>
  

    <script src="https://cpwebassets.codepen.io/assets/common/stopExecutionOnTimeout-2c7831bb44f98c1391d6a4ffda0e1fd302503391ca806e7fcc7b9b87197aec26.js"></script>

  
      <script id="rendered-js">
const cube = document.querySelector("#cube");
const cback = document.querySelector(".back");
const ctop = document.querySelector(".top");
const cleft = document.querySelector(".left");
const cright = document.querySelector(".right");
const glow = document.querySelector(".hexagon");
const powerup = document.querySelector(".powerup");
const transitionTime = "750ms";
let c = 0;

ctop.style.transition = `all ${transitionTime}`;
cleft.style.transition = `all ${transitionTime}`;
cright.style.transition = `all ${transitionTime}`;
cube.style.transition = `all ${transitionTime}`;
powerup.style.transition = `all ${transitionTime}`;
glow.style.transition = `all ${transitionTime}`;
cback.style.transition = `all ${transitionTime}`;

let isOpen = false;
cube.addEventListener("click", openCube);

function openCube() {
  if (!this.isOpen) {
    award();
    ctop.style.transform = "translateY(-3rem)";
    cleft.style.transform = "translateX(-3rem)";
    cright.style.transform = "translateX(3rem)";
    ctop.style.opacity = 0.1;
    cleft.style.opacity = 0.1;
    cright.style.opacity = 0.1;
    cback.style.opacity = 0.1;
    glow.style.opacity = 0.5;
    powerup.style.opacity = 1;
    this.isOpen = true;
    cube.style.animationPlayState = "paused";
    powerup.style.zIndex = 10;
    powerup.style.height = "80px";
    powerup.style.width = "80px";
  } else {
    ctop.style.transform = "translateY(0)";
    cleft.style.transform = "translateX(0)";
    cright.style.transform = "translateX(0)";
    cube.style.opacity = 1;
    this.isOpen = false;
    ctop.style.opacity = 1;
    cleft.style.opacity = 1;
    cright.style.opacity = 1;
    cback.style.opacity = 1;
    glow.style.opacity = 1;
    powerup.style.opacity = 0;
    powerup.style.zIndex = 0;
    cube.style.animationPlayState = "running";
    powerup.style.height = "48px";
    powerup.style.width = "48px";
    changeVar("rgba(255,195,26,0.4)");
  }
}

function changeVar(glow) {
  document.documentElement.style.setProperty("--glow", glow);
}

function award() {
  if (c % 2 == 0) {
    //pp
    powerup.style.backgroundImage = "url('https://cf.quizizz.com/game/img/powerups/lg/power-play.png')";
    changeVar("rgba(69,185,251,0.33)");
  } else {
    // glitch
    powerup.style.backgroundImage = "url('https://cf.quizizz.com/game/img/powerups/lg/glitch.png')";
    changeVar("rgba(246,6,120,0.4)");
  }
  c++;
}
//# sourceURL=pen.js
    </script>

  



</body><grammarly-desktop-integration data-grammarly-shadow-root="true"></grammarly-desktop-integration></html>
