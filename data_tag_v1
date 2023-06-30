function dataTagParseResponse(str) {
    try {
        return JSON.parse(str)
    } catch (e) {
        return {
            body: str
        }
    }
}


function dataTagSendData(data, gtmServerDomain, requestPath, dataLayerEventName, dataLayerVariableName, waitForCookies) {
    dataLayerEventName = dataLayerEventName || false;
    dataLayerVariableName = dataLayerVariableName || false;
    waitForCookies = waitForCookies || false;
    var replaceVariable = function(a, b) {
        return a.replace(/\$\{([^\}]+)\}/g, function(c, d) {
            return b[d] || c
        })
    }
      , sendPixel = function(url) {
        var img = new Image(1,1);
        if (url.startsWith((gtmServerDomain.charAt(gtmServerDomain.length - 1) === "/" ? gtmServerDomain.slice(0, -1) : gtmServerDomain) + "/_set_cookie")) {
            setCookieRunningCount++;
            img.onload = img.onerror = function() {
                img.onload = img.onerror = null;
                setCookieRunningCount--;
                if (xhr.readyState === 4 && dataLayerEventName && dataLayerVariableName && waitForCookies && setCookieRunningCount === 0) {
                    pushToDataLayer()
                }
            }
        }
        img.src = url
    }
      , sendBeacon = function(url) {
        var sendBeaconResult;
        try {
            sendBeaconResult = navigator.sendBeacon && navigator.sendBeacon(url)
        } catch (e) {}
        sendBeaconResult || sendPixel(url)
    }
      , fallbackIterator = function(a) {
        var b = 0;
        return function() {
            return b < a.length ? {
                done: !1,
                value: a[b++]
            } : {
                done: !0
            }
        }
    }
      , pushToDataLayer = function() {
        window[dataLayerVariableName] = window[dataLayerVariableName] || [];
        eventDataLayerData.event = dataLayerEventName;
        window[dataLayerVariableName].push(eventDataLayerData)
    }
      , xhr = new XMLHttpRequest
      , stringifiedData = JSON.stringify(data)
      , response = ""
      , loaded = 0
      , replacements = {
        transport_url: gtmServerDomain
    }
      , eventDataLayerData = {}
      , setCookieRunningCount = 0;
    xhr.open("POST", gtmServerDomain + requestPath);
    xhr.setRequestHeader("Content-type", "text/plain");
    xhr.withCredentials = true;
    xhr.onprogress = function(progress) {
        if (xhr.status === 200 && xhr.responseText.startsWith("event: message\ndata: ")) {
            response += xhr.responseText.substring(loaded);
            loaded = progress.loaded;
            for (var replacedResponse = replaceVariable(response, replacements), nextSeparationPos = replacedResponse.indexOf("\n\n"); -1 !== nextSeparationPos; ) {
                var parsedData;
                a: {
                    var iterableLines;
                    var lines = replacedResponse.substring(0, nextSeparationPos).split("\n")
                      , linesIterator = "undefined" != typeof Symbol && Symbol.iterator && lines[Symbol.iterator];
                    if (linesIterator)
                        iterableLines = linesIterator.call(lines);
                    else if ("number" == typeof lines.length)
                        iterableLines = {
                            next: fallbackIterator(lines)
                        };
                    else
                        throw Error(String(lines) + " is not an iterable or ArrayLike");
                    var eventNameLine = iterableLines.next().value
                      , eventDataLine = iterableLines.next().value;
                    if (eventNameLine.startsWith("event: message") && eventDataLine.startsWith("data: "))
                        try {
                            parsedData = JSON.parse(eventDataLine.substring(eventDataLine.indexOf(":") + 1));
                            break a
                        } catch (e) {}
                    parsedData = void 0
                }
                var event = parsedData;
                if (event) {
                    var sendPixelArr = event.send_pixel || [], i;
                    if (Array.isArray(sendPixelArr))
                        for (i = 0; i < sendPixelArr.length; i++)
                            sendPixel(sendPixelArr[i]);
                    var sendBeaconArr = event.send_beacon || [];
                    if (Array.isArray(sendBeaconArr))
                        for (i = 0; i < sendBeaconArr.length; i++)
                            sendBeacon(sendBeaconArr[i]);
                    if (typeof event.response === "object") {
                        var status = event.response.status_code || 0
                          , body = event.response.body || {}
                          , parsedBody = dataTagParseResponse(body);
                        for (var key in parsedBody) {
                            if (parsedBody.hasOwnProperty(key)) {
                                eventDataLayerData[key] = parsedBody[key]
                            }
                        }
                        eventDataLayerData.status = status
                    }
                }
                replacedResponse = replacedResponse.substring(nextSeparationPos + 2);
                nextSeparationPos = replacedResponse.indexOf("\n\n")
            }
        }
    }
    ;
    xhr.onload = function() {
        if (xhr.status.toString()[0] !== "2") {
            console.error(xhr.status + "> " + xhr.statusText)
        }
        if (dataLayerEventName && dataLayerVariableName) {
            if (!xhr.responseText.startsWith("event: message\ndata: ")) {
                eventDataLayerData = dataTagParseResponse(xhr.responseText);
                eventDataLayerData.status = xhr.status;
                pushToDataLayer()
            } else if (!waitForCookies || setCookieRunningCount === 0) {
                pushToDataLayer()
            }
        }
    }
    ;
    xhr.send(stringifiedData)
}

function dataTagGetData(containerId) {
    window.dataTagData = {
        document: {
            characterSet: window.document.characterSet
        },
        innerWidth: window.innerWidth,
        innerHeight: window.innerHeight,
        screen: {
            width: window.screen.width,
            height: window.screen.height
        },
        dataModel: window.google_tag_manager[containerId].dataLayer.get({
            split: function() {
                return []
            }
        })
    };
    return window.dataTagData
}
