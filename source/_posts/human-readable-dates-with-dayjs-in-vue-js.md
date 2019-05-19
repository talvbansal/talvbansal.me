---
title: Human readable dates with Dayjs in Vue
date: 2019-05-19 18:13:40
tags:
- Laravel
- Javascript
- Ecmascript
- Moment.js
- Day.js
- Dayjs
- Vue
- Vue.JS
- Carbon

autoThumbnailImage: yes
coverImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_1600/v1558287756/posts/stockholm-brunkeberg-tunnel.jpg
coverSize: partial
coverMeta: in
metaAlignment: center
thumbnailImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_280/v1558287756/posts/stockholm-brunkeberg-tunnel.jpg
thumbnailImagePosition: right
---

Recently I blogged about replacing [Moment.js](https://momentjs.com/) with a lightweight alternative [Day.js](https://github.com/iamkun/dayjs).

In Laravel, when working with dates I've been used to using the [Carbon](https://carbon.nesbot.com/) method `diffForHumans`. 

It gives us a date value relative to an optional given date, but it defaults to now. Examples of output from the diffForHumans method looks like:

```
- 5 minutes ago
- One week ago
- Last Year
```

Rather than: 

```
2019-05-19 18:08:40
2019-05-12 18:13:40
2018-05-19 18:13:40
```

We can do the same with Moment.js using the `fromNow` method. Given that Day.js is supposed to have the same API as Moment.js I assumed that the `fromNow` method would "Just work".

Turns out that it didn't. But with some small changes to our code we can get the same `fromNow` method working with Day.js.

<!-- more -->

Day.js has a plugin for working with relative times called "relativeTime".
If you've added day JS using yarn / npm you can just import the relativeTime module, extend Day.js and get to using the new `fromNow` method!

```javascript
import dayjs from 'dayjs';
import relativeTime from 'dayjs/plugin/relativeTime';

dayjs.extend(relativeTime);

const date = '2019-05-13 13:52:15';
console.log(dayjs(date).fromNow());

// 6 days ago
```

To use the `fromNow` method within Vue components, I've been using filters like this:

```html
<template>
    <div>
        {{ date | diffForHumans }}
    </div>
</template>

<script>
import dayjs from 'dayjs';
import relativeTime from 'dayjs/plugin/relativeTime';

export default {
    
    created() {
        dayjs.extend(relativeTime);
    },

    filters: {
        diffForHumans: (date) => {
            if (!date){
                return null;
            }
            
            return dayjs(date).fromNow();
        }
    },
	
    data(){
        return {
            date: "2019-05-13 13:52:15",
        }
    }
}
</script>
```

