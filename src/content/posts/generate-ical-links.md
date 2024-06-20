---
title: JavaScript 日曆事件處理：輕鬆生成 iCal、Google 及 Yahoo 日曆連結
published: 2024-06-20
description: '介紹如何使用 JavaScript 生成 iCal、Google Calendar 和 Yahoo Calendar 的連結'
image: 'https://media.virtualxnews.com/2024/06/486d4c2176dabcb2f0c52d1ff5045a5b.png'
tags: [JavaScript, iCal, Google Calendar, Yahoo Calendar]
category: '前端'
draft: false 
---

身為開發者，我們常常需要處理日曆事件，特別是當網站或應用程式需要讓使用者將事件新增到他們的個人日曆時。這篇文章將介紹如何使用 JavaScript 來建立一個 Calendar 類別，讓您輕鬆生成 iCal 格式的事件資料，以及 Google Calendar 和 Yahoo Calendar 的連結。

## 為什麼需要自訂日曆事件處理？

雖然現代瀏覽器大多支援 iCal 格式，但直接提供 .ics 檔案下載可能會影響使用者體驗。透過生成 iCal 格式的連結，使用者可以直接在瀏覽器中開啟事件，並選擇要新增到哪個日曆服務。


## 程式碼

在下面的程式碼中，包括時間處理和程式碼的結構：

```javascript
import moment from "moment";

/**
 * 產生日曆事件連結的工具類別
 */
export class Calendar {
  /**
   * @param {Object} event - 事件資訊
   * @param {String} event.DTSTART - 開始時間 (格式：YYYYMMDDTHHMMSSZ)
   * @param {String} event.DTEND - 結束時間 (格式：YYYYMMDDTHHMMSSZ)
   * @param {String} event.SUMMARY - 標題
   * @param {String} event.DESCRIPTION - 描述
   * @param {String} event.TZID - 時區 (例如：Asia/Taipei)
   */
  constructor(event) {
    this.event = {
      ...event,
      DTSTART: this.formatTime(event.DTSTART),
      DTEND: this.formatTime(event.DTEND)
    };
  }

  /**
   * 轉換時間格式
   * @param {String} time - 原始時間
   * @returns {String} - 轉換後的時間
   */
  formatTime(time) {
    return moment(time).utc().format("YYYYMMDDTHHmmss") + "Z";
  }

  /**
   * 產生 iCal 格式的文字
   * @returns {String} - iCal 格式的文字
   */
  generateICS() {
    const { DTSTART, DTEND, SUMMARY, DESCRIPTION, TZID } = this.event;
    const icsMSG = `BEGIN:VCALENDAR
VERSION:2.0
BEGIN:VEVENT
DTSTART;VALUE=DATE;TZID=${TZID}:${DTSTART}
DTEND;VALUE=DATE;TZID=${TZID}:${DTEND}
SUMMARY:${SUMMARY}
DESCRIPTION:${DESCRIPTION}
END:VEVENT
END:VCALENDAR`;
    return "data:text/calendar;charset=utf-8," + encodeURIComponent(icsMSG);
  }

  /**
   * 產生 Google Calendar 的連結
   * @returns {String} - Google Calendar 的連結
   */
  generateGoogleCalendarURL() {
    const { DTSTART, DTEND, SUMMARY, DESCRIPTION, TZID } = this.event;
    return `https://calendar.google.com/calendar/u/0/r/eventedit?dates=${DTSTART}/${DTEND}&text=${encodeURIComponent(SUMMARY)}&details=${encodeURIComponent(DESCRIPTION)}&ctz=${encodeURIComponent(TZID)}`;
  }

  /**
   * 產生 Yahoo Calendar 的連結
   * @returns {String} - Yahoo Calendar 的連結
   */
  generateYahooCalendarURL() {
    const { DTSTART, DTEND, SUMMARY, DESCRIPTION, TZID } = this.event;
    return `https://calendar.yahoo.com/?v=60&TYPE=20&VIEW=d&ST=${DTSTART}&ET=${DTEND}&TITLE=${encodeURIComponent(SUMMARY)}&desc=${encodeURIComponent(DESCRIPTION)}&vtz=${encodeURIComponent(TZID)}`;
  }
}
```

## 如何使用 Calendar 類別

```javascript
// 引入 Calendar 類別
import { Calendar } from './path-to-calendar';

// 定義事件資訊
const event = {
  DTSTART: '20240617T120000Z', // 開始時間 (格式：YYYYMMDDTHHMMSSZ)
  DTEND: '20240617T130000Z',   // 結束時間 (格式：YYYYMMDDTHHMMSSZ)
  SUMMARY: '會議',             // 標題
  DESCRIPTION: '團隊會議',     // 描述
  TZID: 'Asia/Taipei'          // 時區
};

// 建立 Calendar 實例
const calendar = new Calendar(event);

// 生成 iCal 連結
const icalLink = calendar.generateICS();
console.log('iCal 連結:', icalLink);

// 生成 Google Calendar 連結
const googleCalendarLink = calendar.generateGoogleCalendarURL();
console.log('Google Calendar 連結:', googleCalendarLink);

// 生成 Yahoo Calendar 連結
const yahooCalendarLink = calendar.generateYahooCalendarURL();
console.log('Yahoo Calendar 連結:', yahooCalendarLink);
```

現在，就有了三個連結：
* icsLink：可以直接在瀏覽器中開啟 iCal 事件。
* googleCalendarLink：在 Google Calendar 中新增事件。
* yahooCalendarLink：在 Yahoo Calendar 中新增事件。

## 結論

透過這個 Calendar 類別，可以更方便地處理日曆事件，讓使用者輕鬆地將事件新增到他們的日曆中。這不僅提升了使用者體驗，也讓網站更具互動性。

:::warning
使用注意事項：
* 在大部分的桌面瀏覽器中，點擊 iCal 連結會下載 .ics 檔案。
* 在 iOS 裝置上，點擊 iCal 連結可能會直接開啟行事曆應用程式，但有時也可能下載檔案。
* 在 Android 裝置上，點擊 iCal 連結通常會下載檔案。
:::