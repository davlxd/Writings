# Implement a Radar with D3 and D3-Force
2019-09-03

![Marvel](/images/marvel-radar.gif)

Inspired by ThoughtWorks [Tech Rdar](https://www.thoughtworks.com/radar),
I always want to create something similiar for my Web resume,
and the four quadrants can be frontend, backend, infrastructure and methodologies,
and each blip represents one of my skills;
the closer it is to the centre, the more familiar I am with it.

After several weeks of hardwork, I managed to create one as the gif shows above,
instead of my actual skills I scrambled Marvel characters and skills for it
just for demonstration purpose.

Here is a wrap-up of some of the key components and steps and tricks,
the code on Github is [here](https://github.com/lxdcn/react-d3-radar-demo),
live demo [here](https://radar-demo.lxd.me/).

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

Data model is pretty simple, each blip contains 4 fields:

  - `quandrant` is which quandrant on the radar this blip belongs to,
  - `name` is the label for this blip
  - `score` is familiarity, the higher the value is, the closer the blip is to the centre
  - `desc` is a more detailed explanation or description will be shown when user click on the blip.

![Marvel](/images/marvel-radar-click-on-blip.gif)


## Draw the background

![Marvel](/images/radar-background.png)

The background looks like a pie with 4 slices, or a cross-section of a log with 3 annual rings.
Therefore for the same reason its implementation is a combination of D3 [pie](https://github.com/d3/d3-shape#pies)
and [arc](https://github.com/d3/d3-shape#arcs).

D3 pie generate the 4 slices (quadrants)

```typescript
const arcs = d3.pie()(new Array(quadrantCount).fill(1))
```

Then for each slice, I used arc to generate SVG paths for annual rings,

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
With D3 Force, you provide the array of nodes you want to position (in my case the blips),
then enforce a set of constraints to them (in D3 terms [forces](https://github.com/d3/d3-force#forces)).

Each force follows its own formula or logic to calculate what's the next position each node should move to.
And with a low iteration rate, you'll be able to see the animation that all the nodes move around driven by forces,
until eventually the positions of all nodes satisify all the forces, to the greatest possible extent averagely.

In my code I mainly use 3 forces:

  - Radial force
  - Within Quandrant force
  - Collision force

[Radial force](https://github.com/d3/d3-force#forceRadial) is included in D3 force module,
it pushes nodes towards the closest point on a given circle, and the radius of the circle is configurable.
So in my case the targeting point is (0, 0) (because I transformed the entire SVG canvas at [here](https://github.com/lxdcn/react-d3-radar-demo/blob/master/src/components/Radar/d3/InitateSvg.ts#L17)),
and radius is in reverse propotion to its `score` value, which can be easily done with the help of D3 scale.

```typescript
const scoreToRadiusScale = d3.scaleLinear()
                             .domain(minAndMaxOfBlipScore)
                             .range([rootSVGRadius, MIN_RADIUS])
```

[Within Quandrant force](https://github.com/lxdcn/react-d3-radar-demo/blob/master/src/components/Radar/d3/ForceOfWithinQuadrant.ts) is a force
I wrote by refering to the source code of [official forces](https://github.com/d3/d3-force/tree/master/src).
It's pretty simple, just drag the nodes back to its quandrant once they exceed the boundaries.
If this force alone takes effect, then most of the blips would move along the axes,
with Collide force and Radial force together then we can push blips toward the centre of quadrants.

[Collision force](https://github.com/d3/d3-force#collision), or Collision avoidance force to be precise,
is also an official force based on [quadtree](https://github.com/d3/d3-quadtree),
it prevents nodes from overlapping each other.
It's circle based and boundary radius is configurable, once some other nodes trespass its circle territory,
this force will drag them apart.

## Collision avoidance for the blip AND its label

Normally the official Collision force should be sufficient for most of the cases,
but it's a little bit tricky for my case because the node that I want enforce collision avoidance
is a symbol plus a text.

![Symbol plus text](/images/symbol-and-text.png)

If I use circles to run collision avoidance, then too much space would be taken and wasted,
especially if this blip has a long label text.
So in my first try I implemented rect-based Collision force by refering to the official circle-based force
and [Rectangular Collision Detection](https://bl.ocks.org/cmgiven/547658968d365bcc324f3e62e175709b) on bl.ocks.org, 
but it didn't work, the blips keep moving back and forth like being electrocuted.
I didn't look into the reason, but I guess it's because the curvature of rectangle is not smooth enough.
Then I moved on to implement a ellipse-based version, took me a while, but still didn't work.
So I thought only circle, whose curvature is constant, based collision avoidance is good enough for an ideal spread animation.

But like I mentioned, circles can take too much space, so I need a better solution.
After slept on this issue for several nights then I came up with this idea.

![With Placeholding Circles](/images/marvel-radar-placeholding-circle.gif)

As you can see, I used a series of placeholding circles, one following another,
sitting on top of text labels, and include them in the Collision force as well,
so they will make sure the text below themselves won't overlap with other symbols and text.

Because of this trick, I had to write extra code to generate placeholding circles, 
extra force to make them sitting on the right position and so on.


## Unfinished work

This project is quite immature, currently works well on Chrominum based browsers and Firefox only.
I hope some of the ideas here can be helpful on your Data Visualization/D3 journey.


