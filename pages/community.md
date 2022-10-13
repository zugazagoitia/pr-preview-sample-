---
layout: default
title: Community
rightmenu: false
permalink: /community
---

{% include notificationBanner.html %}

<h1 class="no-margin-top">Community</h1>

This page lists all of the places to ask for help or discuss Javalin, in order of relevance.

<div class="community-boxes">
  <div class="community-box">
    <h2>GitHub</h2>
    <p>
      The GitHub repo (<a href="https://github.com/tipsy/javalin">https://github.com/tipsy/javalin</a>)
      is where most of the action happens.
      You can create issues to make feature request, to report bugs, or even to ask questions.
      We're not too strict.
    </p>
  </div>
  <div class="community-box">
    <h2>Discord</h2>
    <p>
      Discord is the primary chat server for Javalin. You can use this invite link to join:
      <a href="https://discord.com/invite/sgak4e5NKv">https://discord.com/invite/sgak4e5NKv</a>.
    </p>
  </div>
  <div class="community-box">
    <h2>Slack</h2>
    <p>
      It's not as lively as Discord, but it is a lot more corporate:
      <a href="https://join.slack.com/t/javalin-io/shared_invite/zt-1hwdevskx-ftMobDhGxhW0I268B7Ub~w">https://javalin-io.slack.com/</a>
    </p>
  </div>
  <div class="community-box">
    <h2>Twitter</h2>
    <p>
      Javalin doesn't tweet a lot, but it does retweet and like Javalin related things:
      <a href="https://twitter.com/javalin_io">https://twitter.com/javalin_io</a>
    </p>
  </div>
  <div class="community-box">
    <h2>Stack Overflow</h2>
    <p>
      Javalin has its own tag on Stack Overflow, but it's not a common place to seek help:
      <a href="https://stackoverflow.com/questions/tagged/javalin">https://stackoverflow.com/questions/tagged/javalin</a>
    </p>
  </div>
</div>

{% include contributors.html %}

<style>
  .community-box {
    position: relative;
    color: #444;
    display: block;
    padding: 20px;
    background: #fff;
    border-radius: 5px;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.08);
    margin-top: 24px;
  }

  .community-box h2 {
    font-size: 24px;
    margin-top: 0;
  }
</style>
