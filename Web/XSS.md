#### Non-Intrusive Testing

Try using the <b></b> HTML tag enclosure to print out an example text in bold letters, e.g. <b>this is where I am testing my XSS payload</b>.

This is a non-intrusive way to check if greater than > and less than < characters are allowed to be used. In case your example text is printed **bold**, you have enough evidence that those characters are not getting encoded. You can also check for that in the browser’s developer tools. The reason we are testing for exactly those two characters is that they are needed to insert HTML tags. Being able to add new HTML to a page is still the number one indicator for a potential XSS vulnerability.

![Event Handlers](/docs/assets/images/20210917231205.png)


As we test further we are going to first locate the valid tags that the application allows, followed by finding permitted events and finally by testing payloads.

![Payload Position](/docs/assets/images/20210917231410.png)

Insert <\$\$> Inside the intruder payload section and choose the 'copy tags' option from portswiggers xss cheatsheet. Load the tags and run intruder looking for requests that different from the others.

Insert %20\$\$ to test for events that can be run inside our tags. 



![Event Payloads](/docs/assets/images/20210917231844.png)

back at the cheatsheet copy the events

![Copy Events](/docs/assets/images/20210917231915.png)

run intruder and find a valid event

![Valid Events](/docs/assets/images/20210917231957.png)

At this point we can run javascript inside the event 

![Run Javascript](/docs/assets/images/20210917232030.png)


<iframe width="700" height="400" src="https://www.youtube.com/embed/EoaDgUgS6QA" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
