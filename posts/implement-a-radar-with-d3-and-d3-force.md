# Implement a Radar with D3 and D3-Force
2019-09-03

![Marvel](/images/marvel-radar.gif)

Inspired by ThoughtWorks [Tech Rdar](https://www.thoughtworks.com/radar),
I always want to create something similiar for my Web resume,
so the 4 quadrants can be frontend, backend, infrastructure and methodologies,
and each blip represents one of my skills, the closer it is to the centre,
the more familiar I am with it.
After several weeks of hardwork, I managed to create one as the gif shows above,
instead of my actual skills I scrambled Marvel characters and skills for it
just for demonstration purpose.

Here is a wrap-up of some of the key components and steps and tricks,
the code on Github is [here](https://github.com/lxdcn/react-d3-radar-demo),
live demo is [here](https://radar-demo.lxd.me/).

## Data model

```typescript
interface Blip {
  quadrant: string,
  name: string,
  score: number,
  desc: string,
}

// example:

const blips = [
  {
    quadrant: 'Thanos',
    name: 'Nagging',
    score: 5,
    desc: 'Fun isn\'t something one considers when balancing the universe.'
  },
  {
    quadrant: 'Thanos',
    name: 'Snap finger',
    score: 4,
    desc: 'You\'re strong. But I could snap my fingers, and you\'d all cease to exist.'
  },
  //...
  {
    quadrant: 'Black Widow',
    name: 'Freelancing',
    score: 2,
    desc: 'I\'m multitasking.'
  },
]
```

Data model is pretty simple, each blip contains 4 fields;
`quandrant` is which quandrant on the radar this blip belongs to,
`name` is the label for this blip, `score` is familiarity,
the higher the value is, the closer the blip is to the centre,
`desc` is the more detailed explanation or description will be shown when user
click on the blip.

![Marvel](/images/marvel-radar-click-on-blip.gif)


## Draw the background


## Posision blips


## Collision avoidance for the blip AND its label

