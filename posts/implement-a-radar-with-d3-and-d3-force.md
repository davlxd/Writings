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

![Marvel](/images/radar-background.png)

The background looks like a pie with 4 slices, and a cross-section of a log with 3 annual rings.
Hence for the same reason its implementation is a combination of D3 [pie](https://github.com/d3/d3-shape#pies)
and [arc](https://github.com/d3/d3-shape#arcs).

D3 pie generate the 4 slices (quadrants)
```typescript
const arcs = d3.pie()(new Array(quadrantCount).fill(1))
```
Then for each slice, I used arc to generate svg paths for annual rings,
```typescript
d3.arc()
  .padAngle(padAngleValue)
  .innerRadius(innerRadiusValue)
  .cornerRadius(cornerRadiusValue)
  .outerRadius(outerRadiusValue)
```
and they all overlap each other so the innermost one has the darkest shade.


## Positioning blips

Positioning blips involes another amazing D3 sub-project called [D3 force](https://github.com/d3/d3-force).
With D3 Force, you provide the list nodes you want to position (in my case the blips),
then enforce a set of constraints to them (in D3 terms called [forces](https://github.com/d3/d3-force#forces)).
Each force follows its own formula or logic to calculate what's the next position each node should move to.
With a low iteration rate, you'll be able to see the animation that all the nodes move around driven by forces,
until eventually the positions of all nodes satisify all the forces.

In my code I use 4 forces:






## Collision avoidance for the blip AND its label

