const config = {
  "appSyncApiKey": "da2-wljom62omfbwldbc7cfd2lkyby",
  "appSyncUrl": "https://enoqrts6snd5vkxhq2ziggga3a.appsync-api.us-east-1.amazonaws.com/graphql",
  "pushKey": "BMo09wMd3Q159-S9SR6sVyL7HlwTh0f4IOEvSGLU3NXqcRarstMWLOvyihpRBfpctHbaIxGoVfhASSWwOqPo2To"
};

function shouldPromptForNotifications() {
    if (Notification.permission === "granted" || Notification.permission === "denied") {
        return false;
    }

    const lastDismissTimestamp = parseInt(getCookie("wpn_banner_dismissed"));

    // If the cookie doesn't exist or if 30 days have passed since the last banner dismissal
    return !lastDismissTimestamp || Date.now() - lastDismissTimestamp >= 30 * 24 * 60 * 60 * 1000;
}

const isDesktopBrowser =
    /mozilla|chrome|safari|firefox|edge/i.test(navigator.userAgent) &&
    !/android|webos|iphone|ipad|ipod|blackberry|iemobile|opera mini/i.test(navigator.userAgent);

async function showAndHandleNotificationPrompt(topic) {
    if (shouldPromptForNotifications() && isDesktopBrowser) {
        const swReg = await navigator.serviceWorker.register("/web-push/sw.js");
        const banner = document.createElement("div");
        banner.textContent = 'To sign up for news alerts from Amazon, click here and choose "Allow" for notifications.';
        banner.style.backgroundColor = "#f1f6f6";
        banner.style.textAlign = "center";
        banner.style.padding = "10px";
        banner.style.cursor = "pointer";

        const closeButton = document.createElement("button");
        closeButton.className = "close-button";
        closeButton.textContent = "✕";
        closeButton.style.cssText = `
    float: right;
    background: none;
    border: none;
  `;
        closeButton.addEventListener("mouseover", () => {
            closeButton.style.backgroundColor = "#d0d0d0";
        });
        closeButton.addEventListener("mouseout", () => {
            closeButton.style.backgroundColor = "transparent";
        });
        closeButton.addEventListener("click", (event) => {
            event.stopPropagation(); // This is needed to prevent the parent banner from registering the click and displaying the notification prompt
            banner.style.display = "none";
            document.body.style.marginTop = "";
            // Set a cookie to prevent banner from reappearing for 30 days
            setCookie("wpn_banner_dismissed", Date.now(), 30);
        });

        banner.appendChild(closeButton);

        banner.addEventListener("click", () => {
            Notification.requestPermission().then((permission) => {
                if (permission === "granted") {
                    subscribe(topic, swReg);
                }
                banner.style.display = "none";
                document.body.style.marginTop = "";
            });
        });

        const pageHat = document.querySelector(".Page-hat");
        if (pageHat) {
            pageHat.parentNode.insertBefore(banner, pageHat);
            document.body.style.marginTop = banner.offsetHeight + "px";
        }
    }
}

async function subscribe(topic, swReg) {
    const subscription = await swReg.pushManager.subscribe({
        userVisibleOnly: true,
        applicationServerKey: urlB64ToUint8Array(config.pushKey),
    });

    fetch(config.appSyncUrl, {
        method: "POST",
        headers: { "x-api-key": config.appSyncApiKey },
        body: JSON.stringify({
            query: `mutation($topic: String, $subscription: String) {
                    subscribe(topic: $topic, subscription: $subscription)
                }`,
            variables: { topic, subscription: JSON.stringify(subscription) },
        }),
    });
}

function urlB64ToUint8Array(base64String) {
    const padding = "=".repeat((4 - (base64String.length % 4)) % 4);
    const base64 = (base64String + padding).replace(/\-/g, "+").replace(/_/g, "/");

    const rawData = window.atob(base64);
    const outputArray = new Uint8Array(rawData.length);

    for (let i = 0; i < rawData.length; ++i) {
        outputArray[i] = rawData.charCodeAt(i);
    }
    return outputArray;
}

function setCookie(name, value, days) {
    const date = new Date();
    date.setTime(date.getTime() + days * 24 * 60 * 60 * 1000);
    const expires = "expires=" + date.toUTCString();
    document.cookie = name + "=" + value + ";" + expires + ";path=/";
}

function getCookie(name) {
    const cookieName = name + "=";
    const cookies = document.cookie.split(";");
    for (let i = 0; i < cookies.length; i++) {
        let cookie = cookies[i];
        while (cookie.charAt(0) === " ") {
            cookie = cookie.substring(1);
        }
        if (cookie.indexOf(cookieName) === 0) {
            return cookie.substring(cookieName.length, cookie.length);
        }
    }
    return null;
}

showAndHandleNotificationPrompt("news");
