---
title: Random Likes
date: 2012-07-03 12:21:58
layout: post
---
I recently wanted to help some Facebook pages which make competitions. You like one of their story and they will randomly select one of the people who like it.

It can be very painfull especially to get all the names etc etc. I figured it's very very easy to do it by programmation so here is the code :
<pre lang="php" colla="-"><?php

if(!isset($_GET['story']) || !is_numeric($_GET['story'])) {
    die('USAGE : randomlikes?story=&lt;story id&gt;');
}

$alldata = array();
$url = 'https://graph.facebook.com/'.$_GET['story'].'/likes?limit=5000';
while(($data = json_decode(file_get_contents($url), true)) && !empty($data['data'])) {
    $alldata += $data['data'];
    $url = $data['paging']['next'];
}

$rand = $alldata[array_rand($alldata)];

echo 'The winner is : <a href="http://www.facebook.com/'.$rand['id'].'">'.$rand['name'].'</a>';</pre>

The story id can be found when you click on the date of a post. You have an url like https://www.facebook.com/permalink.php?story_fbid=<story id>&id=<page id>. Take the story_fbid and put it in the url and voilà!

As usual, you can find a demo version here : <a href="http://sd-31170.dedibox.fr/dev/randomlikes/?story=yourstoryid" title="Random likes demo">demo</a>