---
layout: post
title: "Stop nesting animation blocks"
date: 2014-04-01 20:03
comments: false
categories: 
---

Chaining animations together has always been a little bit awkward. You'd do the first step, then in the completion block, do the second step, then in _that_ completion block, do the third step, and so on, until you close it all off with a staircase of doom:

```objc
                    }];
                }];
            }];
        }];
    }];
```

You don't need to do this. 

<!--more-->

New in iOS7, there is a method for building stacks of animations:

`animateKeyFramesWithDuration: delay: options: animations: completion:`

This acts like an ordinary UIView block animation method, with a few extra options, and some magic that happens if you call another method inside the block:

`addKeyframeWithRelativeStartTime: relativeDuration: animations:`.

`addKeyFrame:...` allows you to set a specific sub-animation, with a start time and duration relative to the entire animation block. You'd use this method instead of starting a new animation from the completion block of an earlier one. 

For example, this method moves a view up and down repeatedly, with each movement taking half the time of the one before:

```objc
[UIView animateKeyframesWithDuration:5.0 delay:0.0 options:0 animations:^{        
        [UIView addKeyframeWithRelativeStartTime:0.0 relativeDuration:0.5 animations:^{
            self.verticalPosition.constant = 200.0;
            [self.view layoutIfNeeded];
        }];
        [UIView addKeyframeWithRelativeStartTime:0.5 relativeDuration:0.25 animations:^{
            self.verticalPosition.constant = 50.0;
            [self.view layoutIfNeeded];
        }];
        [UIView addKeyframeWithRelativeStartTime:0.75 relativeDuration:0.125 animations:^{
            self.verticalPosition.constant = 200.0;
            [self.view layoutIfNeeded];
        }];
        [UIView addKeyframeWithRelativeStartTime:0.875 relativeDuration:0.0625 animations:^{
            self.verticalPosition.constant = 50.0;
            [self.view layoutIfNeeded];
        }];
        [UIView addKeyframeWithRelativeStartTime:0.9375 relativeDuration:0.03125 animations:^{
            self.verticalPosition.constant = 200.0;
            [self.view layoutIfNeeded];
        }];
        [UIView addKeyframeWithRelativeStartTime:0.96875 relativeDuration:0.015625 animations:^{
            self.verticalPosition.constant = 50.0;
            [self.view layoutIfNeeded];
        }];
    } completion:nil];
```

The time taken by each part of the animation is relative to the whole duration, and you track the cumulative time to ensure that the start times are right. You can animate multiple views on separate schedules, and also include standard view updates in the main body of the block - these will act like standard animations, running the whole duration. 

## Calculation modes and options

Most of the UIView block animation options are supported (but renamed, since there is a new enum to hold them all). Additionally, there are five calculation modes, which replace the curves from the standard options - if you think about it, curves for the whole animation don't really suit a keyframe-based animation. The modes are:

- **Linear:** makes the animation go from one point to the next, through each point in between.
- **Discrete:** makes the animation leap directly from one point to the next. This option is presumably useful when you are specifying each frame in an animation
- **Paced:** makes the animation "evenly paced", whatever that means. It seems to make it ignore either the start time or duration of the key frames, but not in a predictable way, so I'm not sure why you'd want to do that.
- **Cubic:** uses the [default Catmull-Rom spline](http://en.wikipedia.org/wiki/Cubic_Hermite_spline#Catmull.E2.80.93Rom_spline) (me neither) to interpolate between the keyframe values. Basically this smooths out any rough edges created by your key frame points.
- **Cubic paced:** a cross between Cubic and Paced. 

Linear or Cubic would seem to be the options that suit most use cases. 

A demo project showing the effect of the different calculation modes on the animation described above is available [on GitHub](https://github.com/jrturton/KeyFrameDemo). 