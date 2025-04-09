Raven API Client adalah sebuah _library_ yang dibuat berdasarkan _module_ API Client Module yang terdapat di dalam [Blibli backend framework](https://github.com/bliblidotcom/blibli-backend-framework/tree/master) yang di buat menggunakan Java 8 dan Spring Boot versi 2.3, namun saya _improvisasi_ di Raven API Client dengan menggunakan Java 17 dan Spring Boot versi 3.3 dan 2.7. Improvisasi yang saya buat bisa dibilang tidaklah banyak, karena memang tujuan awal sebenarnya saya membuat API ini adalah untuk mempelajari lebih dalam mengenai _annotation_ dan Spring Boot Webflux lebih dalam. Tetapi seiring berjalannya, saya menyadari ada bagian dari library Api Client Blibli belum _fully reactive_ (setidaknya di versi 0.0.8 yang di open untuk public).

## Test Scenario

Dalam melakukan testing, saya membuat sebuah _project_ Spring Boot sederhana yang memiliki 2 _endpoint_ yang berguna untuk melakukan testing HTTP _request_ GET dan POST. Project yang digunakan identik, hanya terdapat perbedaan di _library_ yang digunakan untuk melakukan API _call_.

Mengingat bahwa _library_ Blibli Backend Framework terakhir di _maintain_ di Java versi 8 dan menggunakan Spring Boot versi 2.3, saya perlu untuk menyesuaikan library Raven API agar mendukung versi tersebut dengan merilis versi 0.1.1-s2 yang menggunakan Spring Boot versi 2.7, sehingga dapat digunakan di Spring Boot 2.7.

Untuk _project_ yang digunakan untuk _testing_ dapat dibuka pada halaman [Github Repository berikut](https://github.com/richard483/raven-benchmark). Terdapat beberapa _branch_ disana, "blibli-client" untuk _project_ yang menggunakan API client dari Blibli, "raven-client" yang menggunakan API client saya, dan "server" sebagai penyedia konten.

### K6 Script

```js
import http from "k6/http";
import { check } from "k6";
import { randomIntBetween } from 'https://jslib.k6.io/k6-utils/1.2.0/index.js';

import { htmlReport } from "https://raw.githubusercontent.com/benc-uk/k6-reporter/main/dist/bundle.js";

export const options = {
  stages: [
    { duration: "10s", target: 10 },
    { duration: "10s", target: 2500 },
    { duration: "30s", target: 2500 },
    { duration: "10s", target: 0 },
  ],
};

export default function () {
  // const url = "http://localhost:8080/demo/hello";
  const randomInt = randomIntBetween(1,10);
  const url = `http://localhost:8080/demo/${randomInt}`;
  const payload = JSON.stringify({
    haha: "you fool"
  });

  const params = {
    headers: {
      "Content-Type": "application/json",
    },
  };

  const res = http.get(url, params);

  check(res, {
    "response code is 200": (r) => {
      return r.status === 200;
    },
    // hello
    // "body is same as expected": (r) => {
    //   return r.body === "HELLO";
    // },

    "body is same as expected": (r) => {
      return r.body === `GOT ${randomInt}`;
    },
  });
}

export function handleSummary(data) {
  return {
    "summary.html": htmlReport(data),
  };
}

```


## Test Result

## To wrap things up

---

**References**

- [https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/)
- [https://id.wikipedia.org/wiki/Daemon](https://id.wikipedia.org/wiki/Daemon)
- [https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/get-started/create-remote-tunnel/](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/get-started/create-remote-tunnel/)
- [https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/do-more-with-tunnels/trycloudflare/](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/do-more-with-tunnels/trycloudflare/)
