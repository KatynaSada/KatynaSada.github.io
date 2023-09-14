---
title: Discovering the mechanism of action of drugs with a sparse explainable network
tags: [XAI, Deep Learning,Drug Response Prediction, MoA]
style: fill
color: primary 
comments: true
description: SparseGO is an effective XAI method for predicting, but more importantly, understanding drug response.
---

<iframe src="https://www.thelancet.com/journals/ebiom/article/PIIS2352-3964(23)00333-X/fulltext" width="100%" height="500px"></iframe>

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>External Content</title>
</head>
<body>
    <div id="external-content"></div>

    <script>
        // Fetch and render the external content
        fetch('https://academic.oup.com/bib/article/24/4/bbad200/7186396')
            .then(response => response.text())
            .then(data => {
                // Create a div to hold the content
                const container = document.getElementById('external-content');
                container.innerHTML = data;
            })
            .catch(error => {
                console.error('Error fetching external content:', error);
            });
    </script>
</body>
</html>


{% include elements/button.html link="https://github.com/KatynaSada/SparseGO" text="View code" %}

{% if page.comments %}
<div id="disqus_thread"></div>
<script>
    /**
    *  RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
    *  LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: https://disqus.com/admin/universalcode/#configuration-variables    */
    /*
    var disqus_config = function () {
    this.page.url = PAGE_URL;  // Replace PAGE_URL with your page's canonical URL variable
    this.page.identifier = PAGE_IDENTIFIER; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
    };
    */
    (function() { // DON'T EDIT BELOW THIS LINE
    var d = document, s = d.createElement('script');
    s.src = 'https://katynasada-github-io.disqus.com/embed.js';
    s.setAttribute('data-timestamp', +new Date());
    (d.head || d.body).appendChild(s);
    })();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
{% endif %}