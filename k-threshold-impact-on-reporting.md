# Impact of k identical reports threshold on reporting

Here, we propose to analyze and discuss the consequences that degrading reporting in exchange for more privacy have on advertisers, publishers, and, ultimately, users. In particular, we tried to estimate the impact of obfuscating some rows in aggregated advertiser/publisher reporting (as proposed by Chrome team here and here), for all parties to see how blind they would be about what happens to and on their properties. In order to do so, we used Criteo proprietary data, aggregated and anonymized. We also share [a python script](https://github.com/Pl-Mrcy/privacysandbox-reporting-analyses/blob/master/k-threshold-impact-on-reporting.ipynb), for everyone to be able to audit and replicate this analysis on their proprietary data.

## Granular reporting is necessary for a safe browsing experience

Privacy is a serious concern from internet users, and we are happy to see that the ecosystem is finally taking notice by trying to offer more transparency. It is very tempting to solve privacy by obfuscating altogether any information a user leaves. However, this could be at the detriment of other equally or more essential considerations from the same users, such as an overall smooth and safe browsing experience. For the sake of a "real world" comparison, nobody would consider turning off the public lights in exchange for streetwalkers to feel that their activities remain private: people would get assaulted, shops would get mugged, and in the end, nobody would want to walk this street ever again. Similarly, if publishers or advertisers are made blind to who prints what and where, users could face similar issues: inappropriate, offensive, or malware-infested advertising, for which neither the publisher nor the advertiser would have the capability to prevent it before it happens, react fast enough once it happened, or take notice at all.

Let us take a classic example of useful reporting that is currently widely used by advertisers and publishers to tackle these kind of considerations. Note that this piece of reporting is not considered to be an extremely granular one by today's standards, and not used to individually track users. 

- For the publishers: see the list of ALL advertisers (redirecting domains) who printed at least one ad on their real estate in the last 24h.
- For the advertisers: see the list of ALL publishers (at least top-level domains) where they printed at least one ad on. 

This level of transparency proves to be particularly useful for actors on both sides, as to the point where some legislation made it a legal obligation. Indeed, disclosing the exhaustive list of placements where ads have been published to the advertiser is mandatory under French law. 

As mentioned above, obvious use-cases for these are brand safety on one side and ad quality on the other. Publishers will want the ads displayed on their property to be aligned with their brand and principles (e.g. as a publisher publishing nutrition guidelines, I don't want to have any soda-related ads). Similarly, brands and advertisers will want their ads to be displayed on digital properties that are aligned with their own brand and principles (e.g. as an air travel company, I do not want my ads to be displayed next to news related to air crashes or climatic catastrophes). And neither would want to ads ruining users experience, either through nefarious content or through malware-infected ads.

Brand safety and ad quality cannot be handled with an "on average" policy. One bad case is enough to go viral and damage advertiser and publisher brands - not to mention the experience of the user that would have been exposed to it. Publishers and advertisers should thus be very reactive and tackle each individual case with care. This is also true of fraud, invalid traffic, and other use-cases which require detailed reporting to be handled appropriately.

## Analysis: estimating the impact of thresholding on different typologies of Publishers and Advertisers

In its various explainers (here and here) about the reporting mechanisms in the privacy sandbox, the Chrome team mentions a threshold k that defines the minimum number of "identical events" for each individual corresponding event to be reported. In Turtledove, this threshold is a key piece to prevent attackers from using reporting to breach user privacy. To this day, however, no value for such threshold has been announced by the Chrome team. We want to estimate on real data the impact such a threshold would have and help set the right value for it based on actual numbers.

This type of threshold mentioned above wouldn't allow the publisher to detect all the advertisers who printed less than k ads during the time period (24h). Let's try to estimate what this misreporting would represent depending on the value of k.

To illustrate the point, we observe the impact on reporting at the daily grain for 4 individual publishers, one major international publisher with large traffic (>2 million daily unique visitors; ~4 million Criteo daily displays), one medium-size publisher (~400k daily unique visitors; ~100k Criteo daily displays) and finally one local publisher (~10k daily unique visitors; ~4k Criteo daily displays) and one very niche publisher (<1k daily unique visitors; ~1k Criteo daily displays).
Similarly, we run the mirror analysis for 3 advertisers, clients of Criteo: one major international retailer (~10 million weekly unique visitors; ~100 million Criteo weekly displays), one medium-size online service provider (~500k weekly unique visitors; ~1 million Criteo weekly displays), and small online business (~10k weekly unique visitors; ~50k Criteo weekly displays).

This analysis is based on Criteo data only. The publishers we are considering have more than one source of demand and the advertisers we are considering may use multiple advertising services (we picked advertisers and publishers we have strong business relationships with, relatively to their size). This analysis remains relevant despite the limited view we have on each actor because this is the way the industry works in practice. The multiplicity of actors on each side is all the more relevant that the analysis shows that the more fractured the supply and demand are, the more crippling the k threshold is for the reporting.
Actors in the industry can find the analysis pseudo-code here and use it to run the same analysis on their proprietary data. We would be very interested to compare the results based on data collected from different typologies of supply/demand and with different products.

### Publisher Reporting

In Turtledove the key aggregation level is the interest group. Bidding, ad rendering, reporting are all considered at this level. Here, for the sake of simplicity, we consider the "advertisers" rather than the interest groups. In reality, advertisers will want to refine their targeting and will define hundreds to thousands of interest groups, making interest groups a hundred or a thousand times smaller than the advertiser itself.

The exact question we are trying to answer running this analysis is the following: as a publisher, what would be the share of unique interest groups (here advertisers) which would go unreported in the daily reporting depending on the value of the threshold k?

You can find the results in this plot below: Each colour corresponds to a different publisher, representative of a "class of publishers" we currently work with.

![Publisher Report Share of Advertisers](https://github.com/Pl-Mrcy/privacysandbox-reporting-analyses/blob/k-threshold-impact-on-reporting/src/k-threshold-impact-on-reporting/20200630-publisher-report-nadvertisers.png?raw=true)

Interpretation: with a threshold k=10, this major international publisher would have between 20% and 25% of advertisers making displays on its property unreported. This number grows to more than 50% for k = 100 and more than 80% for k = 1000.

![Publisher Report Share of Displays](https://github.com/Pl-Mrcy/privacysandbox-reporting-analyses/blob/k-threshold-impact-on-reporting/src/k-threshold-impact-on-reporting/20200630-publisher-report-ndisplays.png?raw=true)

Interpretation: with a threshold k=100, this major international publisher would have between more than half of advertisers making displays on its property unreported as shown in the upper plot. However, this represents an extremely small share in terms of numbers of displays (~2%).

The share of unreported advertisers is rapidly growing as k increases. This is particularly true for smaller publishers.

This data is only based on Criteo Data. In reality, the profile would look very similar, but the absolute numbers behind would be bigger as publishers usually use several sources of demand.

### Advertiser Reporting

Again for the sake of simplicity, we equate publisher to top-level domains. In reality, advertisers sometimes require a much finer grain to avoid certain website subdomains and some specifics URLs. Indeed, a travel agency may be perfectly ok with printing on a general news website but not on the travel accidents subsection.

The exact question we are trying to answer running this analysis is the following: As an advertiser, what would be the share of the unique top domains which would go unreported in the daily reporting depending on the value of the threshold k?

![Advertiser Report Share of Publishers](https://github.com/Pl-Mrcy/privacysandbox-reporting-analyses/blob/k-threshold-impact-on-reporting/src/k-threshold-impact-on-reporting/20200630-advertiser-report-npublishers.png?raw=true)

Interpretation: with a threshold k=10, a major advertiser we work with would have more than 80% of the publishers he printed displays on unreported. Setting this threshold at 2 (k=2) already filters our close to 60% of the publishers from the daily reporting.

![Advertiser Report Share of Displays](https://github.com/Pl-Mrcy/privacysandbox-reporting-analyses/blob/k-threshold-impact-on-reporting/src/k-threshold-impact-on-reporting/20200630-advertiser-report-ndisplays.png?raw=true)

Here again, we see that the share of reported unique publishers on which the advertiser printed at least one ad decreases when k increases.

The advertiser perspective is even more striking: even for major advertisers, a low value of k would already lead to a majority of the publishers they printed ads on to go unreported. Comparing the two plots above, it becomes clear that advertisers, no matter their size, print on a very long tail of small publishers. They print daily very few ads daily on a large number of them.

## Conclusion

While impacting particularly severely the smaller actors on both sides, we see that even the major stakeholder would receive very partial reporting, even for low k values. This hole-filled reporting would definitely not meet the requirements of ad quality and brand safety use-cases, nor the security or fraud investigation use cases, like many others.

Ultimately, while aiming at preserving users privacy, we believe reporting needs to also take into account these other important aspects for an optimal user web experience. There are fortunately other reporting schemes available that could help us combine an acceptable level of reporting and user privacy! We think we should invest in these rather than pushing this one further.

<hr>

**The pseudo-script used ot run this analysis his available [here](https://github.com/Pl-Mrcy/privacysandbox-reporting-analyses/blob/master/k-threshold-impact-on-reporting.ipynb)**