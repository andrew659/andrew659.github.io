---
layout: post
title: Android Hybrid App Research Papers
date: '2016-12-09 16:59'
categories:
  - Android
  - research-paper
---

<style>
table {
	color:#333333;
	border-width: 1px;
	border-color: #666666;
	border-collapse: collapse;
  margin-bottom: 10px;
}
th {
	border-width: 1px;
	padding: 8px;
	border-style: solid;
	border-color: #666666;
	background-color: #dedede;
}
td {
	border-width: 1px;
	padding-left: 10px;
  padding-right: 10px;
  padding-top: 2px;
  padding-bottom: 2px;
	border-style: solid;
	border-color: #666666;
	background-color: #ffffff;
}
</style>

This post lists and discusses interesting research papers related to Android hybrid apps.

### What is a hybrid app?

There is no standard definition yet. The most precise one I can find is from a [WAMA 16](http://dl.acm.org/citation.cfm?id=2993263) paper by Ali et al.. Currently, there are three popular ways to build mobile apps.

1. `Native apps`: Developers use SDKs and frameworks for the targeted platform to build apps.

2. `Moible web apps`: Developers use web technologies such as HTML, CSS, and Javascript to build an app as one website that is optimized for mobile devices.

3. `Hybrid apps`: Sitting in between native and web apps, hybrid apps bridge the gap between the above two methods. This method uses a common code base to simultaneously deliver native like apps to multiple platforms.

Hybrid apps are usually created using Cross Platform Tools (CPTs) with two approaches:

1. The first allows developers to use web technologies (HTML, CSS, and Javascript) to create a code base, which runs in an internal browser (_WebView_) that is wrapped in a native app. Example CPTs: [PhoneGap](http://phonegap.com/), [Trigger.io](https://trigger.io/).

2. The second approach allows developers to write code in their familiar language such as C#, Ruby and then the code gets compiled (or transformed) to native code for each platform. Example CPTs: [Xamarin](https://www.xamarin.com/), [Appcelerator Titanium](http://www.appcelerator.com/), and [Adobe Air](https://get.adobe.com/air/).

### Hybrid apps vs. native and web apps

Generally speaking, hybrid apps have the following characteristics when comparing with native and web apps ([data source](http://baike.baidu.com/view/8488720.htm)).

 |  | Web apps  | Hybrid apps  | Native apps |
|:--:|:---:|:---:|:--:|
|Development cost  |  Low | Medium  | High |
| Maintenance | Easy  | Easy  | Complex |
| User experience | Bad  | OK  | Good |
| Available in app store?  | No  | Yes  | Yes |
| Installation | No need  | Needed  | Needed |
| Cross platform? | Yes  | Yes  | No |

### Research papers

---

[**1. Mining and characterizing hybrid apps**](http://dl.acm.org/citation.cfm?id=2993263). This paper studied 15,512 hybrid apps developed with PhoneGap, Appcelerator Titanium, and Adobe Air and observed several findings.

- While hybrid apps are becoming popular, native apps still dominate the market place due to their competitive advantage in terms of performance and supported features.

- The PhoneGap CPT is the most popular CPT and widely-used to develop business, lifestyle, and travel & local apps.
- Despite the least popularly used CPT, Adobe Air produces apps that are downloaded and reviewed much more by users. The main reason might be Adobe Air is widely used to develop games.

- In 17 out of the 25 app categories, the user perceived ratings of hybrid apps is even higher than native apps, indicating that it is possible to create a successful hybrid app that can compete with their native counterparts.

---

[**2. HybriDroid: static analysis framework for Android hybrid applications**](http://dl.acm.org/citation.cfm?id=2970368).

to be continued.

### References

1. Sungho Lee, Julian Dolby, and Sukyoung Ryu. 2016. HybriDroid: static analysis framework for Android hybrid applications. In Proceedings of the 31st IEEE/ACM International Conference on Automated Software Engineering (ASE 2016). ACM, New York, NY, USA, 250-261.

2. Mohamed Ali and Ali Mesbah. 2016. Mining and characterizing hybrid apps. In Proceedings of the International Workshop on App Market Analytics (WAMA 2016). ACM, New York, NY, USA, 50-56.
