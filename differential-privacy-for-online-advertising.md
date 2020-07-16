# Differential privacy for online advertising

Differential privacy is a strong, mathematical definition of privacy in the context of statistical and machine learning analysis. It is often presented as a way to protect user privacy, with a tradeoff with regards to the utility of the information returned. 

In this document, we propose to discuss and showcase why differential privacy is not adapted to some major online advertising use-cases, as using it will lead to a mostly unusable reporting. We attach to this article a [python notebook](https://github.com/Pl-Mrcy/privacysandbox-reporting-analyses/blob/differential-privacy-for-online-advertising/differential-privacy-for-online-advertising.ipynb) allowing other actors in the industry to play with their own data and generate differential private reports and witness the impact by themselves.

All examples below, illustrating our points, are based on anonymized Criteo proprietary data.

## Differential privacy inadequacy

The idea behind differential privacy is to add noise to the result of each information given to the database querier, such that it is impossible to distinguish the results between two databases differing by one person (meaning that if a single person is included or removed from the database, the results should be roughly the same). More details on differential privacy can be found [here](https://privacytools.seas.harvard.edu/files/privacytools/files/pedagogical-document-dp_new.pdf) 

The level of privacy is controlled by a parameter called the privacy budget, or epsilon. The lower the epsilon, the higher the privacy guarantee and the higher the noise level. From the previously shared document, epsilon should be between 0.0001 and 1, with 1 being the value with the best for reporting accuracy, and worst for user privacy.

There are two main reasons why differential privacy is not well suited for web advertising:
- Some labels of highest interests (click, sales, leads, etc.) are very rare, meaning that individual users have huge relative impacts on such metrics. For instance:
	- The probability of an ad to be clicked is about 0.5%.
	- An ad leads to a post-click sale approximately once every 10,000 displays.
- Features of interest dimensionality. Because of the web structure, online advertising data is of very high dimensionality / cardinality.
	- There are millions of URL domains people currently do ads on. On such domains, there are even more subdomains that might be of interest (yahoo finance ads differs from yahoo news).
	- The display size adds another layer of dimensionality.
	- A lot of others high cardinality features (e.g. localisations, etc.) are also of high interest.

## Impact of differential privacy on reports

By design, differential privacy adds a level of noise that is absolute. It does so so that an external reader can't tell the difference if an individual is included or removed from the reports. 

This means that if one asks for the number of users exposed to an ad, the differential private noise will be the same order of magnitude if there are 10 users or 10,000 (Let's assume we are taking an epsilon of one, which is somehow equivalent to add or remove randomly one to the actual query result).  This means that for the advertiser with 10 displays, the relative noise will be 10% (one over 10), and for the one with 10,000, it will be 0.01% (one over 10,000). 

This absolute noise level is one of the main reason for differential privacy success in sectors outside of ad tech. This means that, usually, with a big database, the level of noise is very small relatively because the difference between 10,000 exposed users or 10,001 users is irrelevant for all stakeholders.

But for online advertising, it is different.

### Label rarity impact

In our previous example, differential privacy noise was irrelevant, because we considered the number of displays, and indeed a noise of 1 is irrelevant for protecting the information transmitted by the report. But with 10,000 displays, there is usually 50 clicks and only one sale. As the added noise is absolute, this means that the noise on the number of clicks will be of plus or minus one click and plus or minus one sale for the number of sales.

This means that for clicks, the relative level of the noise is already 2% (one over 50), and **for sale, it is 100% (one over one)**. 

To get an error of less than 2% with our example on the conversion rate, one would need 500,000 displays (and this level of noise is actually considered as quite low for differential privacy, meaning a low level of privacy protection)!

### Dimensionality impact

Online advertising often aims at providing sales and visits to the advertiser. However, the level of CTR and CR differs widely across publishers, formats, and user interests.

A user interested in shoes is significantly more likely to click on an ad related to shoes, but it will change a lot depending on the context. For instance, the likelihood of user interaction depends a lot on:
- The publisher page content. If the content is in line with the user interest, it might magnify the chance of a click and a sale, if completely unrelated, it might reduce it.
- The ad size and position. Some ads are visible only if the user scrolls down. These ads are less noticed and therefore less valuable. Same thing for the ad size.

It was the ability to use all this information together to know the fair price for an ad that leads advertising to support publisher revenue as it did for the last 10 years.

How does this relate to differential privacy? This means that having reliable reports on all these dimensions together, the absolute level of noise needs to be added to all these dimensions.

And if some advertisers may have more than 500,000 ads displayed and therefore may be able to measure their CR with less than 2% noise almost none of them has 500,000 ads on most of the domains they are doing advertising on, and even less on a given domain cross a given ad size. 

This means that it won't be possible for an advertiser to chose performing inventory over non-performing inventory, as it won't be possible for the advertiser to differentiate between them!

It will probably lead some bad-faith actors to start adding invisible or very low-quality inventory on sale, as no actor will be able to spot it and it will be impossible to differentiate between these bad-faith actors and good faith actors, leading to a race to the bottom as good faith actor would see their revenue plummet, leading them no choice but to follow. This mechanism has been well described by the [seminal paper a market for lemons](https://en.wikipedia.org/wiki/The_Market_for_Lemons). 

Because of both the rarity of some labels of interest and the extremely high dimensionality of the web, we think that differential privacy is not a good framework for all reporting. We will give a real example based on our data to show the effect of differential privacy on our reports.

## Real-life examples

Here we want to showcase the impact of differential privacy on typical daily reporting.

A notebook is available to run this kind of mock reports on your own data and see the impact of differential privacy.

### Small Advertiser

The following graphs show the evolution over time of 4 KPIs. You can compare the real data (as currently shown in reporting) with the differential private report a small advertiser would receive. Differential private epsilon is set at 1 (please remember that 1 is the best value possible for reporting in the range we consider), and Laplacian noise is added (details on additive noise for differential privacy can be found [here](https://en.wikipedia.org/wiki/Additive_noise_mechanisms)). The line in orange is the differential report, in blue the actual KPIs.

![Publisher Report Share of Displays](https://github.com/Pl-Mrcy/privacysandbox-reporting-analyses/blob/differential-privacy-for-online-advertising/src/differential-privacy-for-online-advertising/20200715-diffentialprivatereport-vs-actualdata.png?raw=true)

We see very well the 'Label rarity impact' for this advertiser. The displays are well reported as a single user has a very small impact on the result. For clicks (and therefore CTR), if the shape of the evolution is mostly represented, the noise is already very significant. On sales is very hard to do anything with the differential private report, as the noise level is as high as the signal. This is because a single user may do a sales and that the absolute number of sales is very low, so the relative level of differential private noise is very high.

This is an issue because this means that it will be extremely hard to pilot a small advertiser campaign, giving an unfair advantage to the bigger actors.

### Large Advertiser

In order to further illustrate our point, the report of one of Criteo's biggest advertiser will be used. Three main metrics were considered:
- The number of displays
- The number of clicks
- The number of sales 

In this example, an epsilon of 1, the highest in the range was chosen. Laplacian noise was used in order to respect differential privacy. The highest the epsilon, the more reliable the report and the less private the report. This is, yet again, a best-case scenario for the advertiser.

We projected the report per publisher domain in order to show both the dimensionality and label rarity impact. Publisher #1 is the biggest publisher in term of displays, Publisher #100 the 100th biggest. There were around 3,000 publishers in this report.

<table>
    <thead>
        <tr>
            <th rowspan=2>Publisher</th>
            <th colspan=3>Displays</th>
            <th colspan=3>Clicks</th>
            <th colspan=3>Sales</th>
        </tr>
        <tr>
            <th>Actual Value</th>
            <th>Turtledove value</th>
            <th>Difference (%)</th>
            <th>Actual Value</th>
            <th>Turtledove value</th>
            <th>Difference (%)</th>
            <th>Actual Value</th>
            <th>Turtledove value</th>
            <th>Difference (%)</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Publisher #1</td>
            <td>2,003,416</td>
            <td>2,003,448</td>
            <td>0.00%</td>
            <td>4,862</td>
            <td>4,856</td>
            <td>-0.12%</td>
            <td>200</td>
            <td>199.4</td>
            <td>-0.29%</td>
        </tr>
        <tr>
            <td>Publisher #10</td>
            <td>190,128</td>
            <td>190,126</td>
            <td>0.00%</td>
            <td>1,167</td>
            <td>1,165</td>
            <td>-0.12%</td>
            <td>267</td>
            <td>270</td>
            <td>1.26%</td>
        </tr>
        <tr>
            <td>Publisher #50</td>
            <td>37,504</td>
            <td>37,502</td>
            <td>-0.01%</td>
            <td>153</td>
            <td>155</td>
            <td>1.25%</td>
            <td>8</td>
            <td>10.2</td>
            <td>27.47%</td>
        </tr>
        <tr>
            <td>Publisher #100</td>
            <td>10,628</td>
            <td>10,623</td>
            <td>-0.05%</td>
            <td>22</td>
            <td>20.9</td>
            <td>-5.14%</td>
            <td>11</td>
            <td>10.4</td>
            <td>-5.42%</td>
        </tr>
    </tbody>
</table>

On label rarity:
- On all publishers, the number of displays metric is well preserved. In this case, differential privacy allows for very reliable reports and protection of the user privacy.
- The click metric is reliable for Publisher #1 and Publisher #10 and starts to be noisy for Publisher #50 and Publisher #100.
- The sale metric is reliable for the biggest publisher but is more and more unreliable as the number of sales goes downs. For Publisher #50, the error is 25%, the report is not reliable. 

This example shows that even for one of the biggest Criteo's advertiser, a differential private report will start to be unreliable when considering sales. 

This report only breaks down the data by publishers. To efficiently run an advertising campaign, a report should not only breakdown by publishers but also sizes, devices, etc.

The following table shows the report when projecting on all those essential dimensions. Modalities are ordered by display count. Results for the first, the tenth, the hundredth and the thousands most frequent modalities are shown.

<table>
    <thead>
        <tr>
            <th rowspan=2>Nth most frequent modality</th>
            <th colspan=3>Displays</th>
            <th colspan=3>Clicks</th>
            <th colspan=3>Sales</th>
        </tr>
        <tr>
            <th>Actual Value</th>
            <th>Turtledove value</th>
            <th>Difference (%)</th>
            <th>Actual Value</th>
            <th>Turtledove value</th>
            <th>Difference (%)</th>
            <th>Actual Value</th>
            <th>Turtledove value</th>
            <th>Difference (%)</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>1st</td>
            <td>443,938</td>
            <td>443,958.7</td>
            <td>0.00%</td>
            <td>2,272</td>
            <td>2,273.5</td>
            <td>0.07%</td>
            <td>381</td>
            <td>382.2</td>
            <td>0.30%</td>
        </tr>
        <tr>
            <td>10th</td>
            <td>122,359</td>
            <td>122,350</td>
            <td>-0.01%</td>
            <td>231</td>
            <td>233.7</td>
            <td>1.16%</td>
            <td>28</td>
            <td>28.6</td>
            <td>2.15%</td>
        </tr>
        <tr>
            <td>100th</td>
            <td>16,101</td>
            <td>16,093.4</td>
            <td>-0.05%</td>
            <td>37</td>
            <td>40.8</td>
            <td>10.27%</td>
            <td>3</td>
            <td>3.6</td>
            <td>21.40%</td>
        </tr>
        <tr>
            <td>1000th</td>
            <td>319</td>
            <td>340</td>
            <td>6.64%</td>
            <td>3</td>
            <td>5.2</td>
            <td>73.8%</td>
            <td>0</td>
            <td>0.7</td>
            <td>NA</td>
        </tr>
    </tbody>
</table>

We see that for clicks, even the 100 most frequent modality starts to be unreliable. For the 1,000th, clicks are widely inaccurate, and sales are only noise. There is still around 30% of the displays and the sales that are done on modality rarer than the 1,000th most frequent (and 50% for modalities rarer than the top 100), meaning that for these, the report will be completely unreliable.

## Conclusion

We have seen that a daily reporting would be significantly impaired if differential privacy was to be used, even for one of the biggest Criteo's advertiser. This goes from bad to worse when the stakeholders become smaller.

Differential privacy could be used for specific advertising use-cases (such as spend management) and provide utility but won't provide a "one size fits all" solution.

The cardinality of some dimensions is such that there won't be way around it. However, for basic use-cases and in order to reduce the noise, one option is to increase the time period you consider, in order to increase the volumes. Advertisers could find a trade-off between the accuracy of the report and the frequency at which they can receive it. The right trade-off would be specific for each KPI, in alignment with each use-case: daily for the number of displays (campaign spend management), weekly for clicks (CTR computation), monthly for sales (Post-click sales performance). A follow-up to this analysis would be to estimate the order of magnitude to consider for each KPI, in order to remain in a pre-defined range compared to the real measure.

<hr>

**The pseudo-script used to run this analysis is available [here](https://github.com/Pl-Mrcy/privacysandbox-reporting-analyses/blob/differential-privacy-for-online-advertising/differential-privacy-for-online-advertising.ipynb)**

<hr>

_Basile Leparmentier_  [b.leparmentier@criteo.com](mailto:b.leparmentier@criteo.com)    
_Dorian Bilinski_  [d.bilinski@criteo.com](mailto:d.bilinski@criteo.com)    
_Paul Marcilhacy_  [p.marcilhacy@criteo.com](mailto:p.marcilhacy@criteo.com)
