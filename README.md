# Color Swatches

Simply install dependencies and run.

## Project Setup and Execution

Install dependencies
```sh
npm install
```

Launch project
```sh
npm run dev
```
The project may then be viewed at the default vite local url: http://localhost:5173/

All of the meaningful code is within `src/App.vue`


## The Challenge
The obvious hurdle of this challenge is the fact that the only way to find all the defined names for a given saturation and lightness of an hsl color is to make every request for each possible hsl value. There may be a way to calculate distances between names more effieciently using the hex values, but considering the complications with calculating [between hex and hsl](https://mohanvadivel.com/thoughts/color-model-conversion) and the existance of this challenge, I figured managing multiple requests is the game.

## Design Decisions
I will be honest, I changed my approach a few times. Initially prototyping this, I knew it would be best to make parallel requests. Making all the requests at once may have worked locally and felt the most powerful, but it was far from sustainable considering realistic API and browser limitations. The next move was to batch the requests. Since http/2 can handle 100 concurrent requests on modern browsers, I wanted to stay close to that number. I could do a batch of 60 requests, but I was still facing loading experiences as the requests started from the lowest hue value. 
The real secret sauce was spreading a batches' requests across the entire hue scale, and filling the gaps with followup request batches. 
Such that in the first batch of requests, hue values of `[0, 15, 30, ..., 360]` are requested first.

This works particularly well, since "Red" for instance may occupy 4 hue units, while "Scarlet" takes 6 and so on. By spreading out the requests, the application has a better chance to hit more distinct names on the first request batch. 

## Further Improvements
While not implemented, if I took more time on this, I would ensure a single request is followed-up with an immediate new request. Right now, a batch isn't started until all the requests in the previous batch are finished. This means, near the end of a batch, there's a chance that only a few requests are still running. Assuming that I am making 72 requests in a batch, I could maintain that number of concurrent requests if new requests are waiting on a previous individual request, not all the requests in the previous batch. Luckily we're fighting over milliseconds in this situation, but on a large-enough scale, that matters. 