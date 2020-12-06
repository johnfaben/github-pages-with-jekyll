---
title: "FINDING_CARDS"
date: 2020-12-06
---

The output of the ML model is a series of predictions, each one consisting of a card name, an x coordinate, a y coordinate and a confidence level for the prediction. The full output of the prediction model for the image below is [here](/ocr/all_predictions_2.csv) Below I've just included the predictions for a single card. 

<table class="table table-bordered table-hover table-condensed">
<thead><tr><th title="Field #1">card</th>
<th title="Field #2">confidence</th>
<th title="Field #3">x</th>
<th title="Field #4">y</th>
</tr></thead>
<tbody><tr>
<td>QH</td>
<td align="right">36.44</td>
<td align="right">669.312</td>
<td align="right">1633.68</td>
</tr>
<tr>
<td>QH</td>
<td align="right">99.01</td>
<td align="right">2128.632</td>
<td align="right">2408.76</td>
</tr>
<tr>
<td>QH</td>
<td align="right">99.74</td>
<td align="right">2123.664</td>
<td align="right">2406.384</td>
</tr>
<tr>
<td>QH</td>
<td align="right">99.61</td>
<td align="right">2149.296</td>
<td align="right">2719.008</td>
</tr>
<tr>
<td>QH</td>
<td align="right">99.73</td>
<td align="right">2123.016</td>
<td align="right">2408.112</td>
</tr>
</tbody></table>

The model is very confident of some predictions - these correspond to the actual QH, which you can see is in the West hand. It also has one low confident prediction (the first one) that the card is in another hand. This corresponds to the actual location of the HK in the South hand. 

<img src="/images/2.jpg"> 

The output we actually want is a simple list of card and which hand they are in, which we can then easily convert into whatever format we want (e.g. pbn/json/etc.). The intermediate step is a list of cards with our best guess as to that card's location. Once we have this, we can relatively easily calculate which hand each card is in (the current approach is to use k-means clustering, although this is probably actually overkill, and we could hard-code something if needed, it works for now). 

So, how do we get from the list of predictions with various levels of confidence to a list of cards with locations? Let's have a look at what we get with the simplest possible logic.

<h4>Just take the highest confidence prediction</h4>

Here's the pbn we get from the above image with this logic: 

[Deal "N:5.743.8742.AT8752 Q943..AQJ95.J643 .K9865.T3.K KJ8762.AQJ2.K6.9]

Let's just look at one card which is misplaced, the 5C:

<table class="table table-bordered table-hover table-condensed">
<thead><tr><th title="Field #1">confidence</th>
<th title="Field #2">x</th>
<th title="Field #3">y</th>
<th title="Field #4">card</th>
</tr></thead>
<tbody><tr>
<td align="right">0.777</td>
<td align="right">1488.52</td>
<td align="right">509.11199999999997</td>
<td>5C</td>
</tr>
<tr>
<td align="right">0.9265</td>
<td align="right">2541.984</td>
<td align="right">2274.552</td>
<td>5C</td>
</tr>
<tr>
<td align="right">0.8511</td>
<td align="right">2547.096</td>
<td align="right">2274.3360000000002</td>
<td>5C</td>
</tr>
<tr>
<td align="right">0.2654</td>
<td align="right">2505.408</td>
<td align="right">2744.928</td>
<td>5C</td>
</tr>
<tr>
<td align="right">0.7665</td>
<td align="right">2505.408</td>
<td align="right">2744.928</td>
<td>5S</td>
</tr>
</tbody></table>

The highest confidence prediction for the location of the club 5 is in the same place as the highest confidence prediction for the spade 5.
There are some other cards where there are just no predictions at all for the location of that card. (e.g. the QH, the AS and the TS). 

There are some things we can try to work around these sort of issues - e.g. if we have the location of most of the cards (or if there are only cards missing from one hand), we could infer the location of the missing cards. Similarly, we might be able to build some logic that relies on the fact that it's not possible for two cards to be in the same place. One difficulty with the latter is that while the boxes overlap, they're not identical.
