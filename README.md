CORS should be Abandoned, Adobe’s Approach should be Adopted
20171013
Change history:
draft #1 20171012 http://lists.w3.org/Archives/Public/public-webapps/2017OctDec/0035.html
20171013 this version with some typo corrected, and some minor editing, a better presentation is http://blog.sina.com.cn/s/blog_93b70ae70102wxe8.html 


The Original Purpose of CORS(Cross Origin Resources Sharing)

Concerning "CORS", its original purpose is to relax the same site(origin) policy to meets web developers’ need: When http://1st.com/a.html is being loaded in a browser, a.html requires a piece of data from http://2nd.com/tickerMSFT or secreteData. Let me stop use Travis' evil.com & popularbank.com, I am afraid that some people stop thinking clearly when they see the word “evil”. It seems the word stir up their emotions. 

In the old days, browsers do not allow a.html from http://1st.com/ to load any data from http://2nd.com/ because of the same site policy imposed by browsers. Since code is allowed, so web developers got around it to load data by embedding the data into javascript code. Illustrated as follows.
if http://2nd.com/tickerMSFT contains

{timestamp:1234234,bid=70,ask=80}

we change its content to
var vname={timestamp:1234234,bid=70,ask=80};

then when it is loaded with a script element in a.html, a.html can access vname, that’s the data it needs. This is the script element approach.

or we change its content of tickerMSFT to
functionName('{timestamp:1234234,bid=70,ask=80}');

where functionName is a function defined in a.html, when the above code is loaded, the function is executed, and a.html have access to the data. This is called jsonp approach since the functionName is padded to the request made to http://2nd.com/tickerMSFT.

Either approach works, but they brought security issues, since whatever is loaded is executed. While the original needs of a.html is to load data, not code, especially a.html does not want whatever loaded from other sites to be utilized before examined. So we need browsers to relax the same site policy to allow a.html to load anything from other sites in a secure manner: we hoped there is a mechanism for a.html to instruct the browser to deem other sites as same origin, so data can be loaded as needed. Concerning codes from other sites, I will discuss it in other threads.

Without browser support, to ensure a.html to get what it needs in a secure manner, the only way is to load http://2nd.com/tickerMSFT at server http://1st.com/ then, after being checked, serve a.html the data. This approach conforms the same site policy imposed by browsers, though web developers do not like this approach.

The above is the situation before the first version of CORS is released. The context is that 1st.com needs the public resources on other sites such as 2nd.com, but do not trust them completely. Please focus on this, and do not act like many people to mess around and to dream about the securities of those other sites. If you really care, I will talk about it later.

People Went Off Road - CORS is not the solution

People like Florian Bosh & Tab Atkins were thinking how should they instruct browsers to distinguish
requests for public data such as tickerMSFT from requests for secreteData. They thought if a.html is allowed to load tickerMSFT from 2nd.com, then it can also load secreteData from 2nd.com. I guess the authors of CORS(Matt Oshry, Brad Porter, J Auburn, and Anne Van Kesteren) must think in same way, so those people devised a domain based referrer authorization check mechanism (CORS) to be imposed by browsers. 

The mechanism requires authorization rules (access control) to be defined for all resources and the authorization rules are mainly imposed by browsers. For example, the authorization rules on secreteData is declared for browsers to allow access from some sites with some specific methods (1st.com might not be in the list), and rules (access control) on publicData such as tickerMSFT is declared for browsers to allow access from all domain (Access-Control-Allow-Origin "*") with any method.

I think they went too far and off road (while I can only see 1 step ahead) from what we are looking for. Clearly, the authorization check is the responsibility of 2nd.com, not of browsers. Tab Atkins(post #0015) & Florian Bosh(post #0017 and #0025) thought it is the responsibility of browsers. Florian Bosh & Tab Atkins never asked how the manager of 2nd.com can distinguish requests referred by http://1st.com from requests referred by https://2nd.com/entrypoints. I would guess they know, and I guess we all know, so let me omit this. Since the designer of 2nd.com can and must do authorization check (referrer based, user or role based, time base, etc) on resource access, at least on access to secreteData, there is no need for 2nd.com to delegate authorization check from their own web server to browsers. And no security wary manager will allow the authorization check of their protected resources to be dependent on browsers.

The Simple Adobe’s Solution Should be Adopted

To meet the cross origin needs of web developers from then and now, the simplest solution is for http://1st.com/a.html to instruct browsers to deem http://2nd.com/ as same origin either with <meta name="allowOrigin" content="http://2nd.com"/> or with an http header(I would suggest to allow multiples). This directive mechanism is asymmetric since this directive does not imply http://2nd.com/ also deems http://1st.com/ as same origin.

As long as browsers can take this into same origin (I use term “origin” since it is not just one site) check procedure, then our original purpose is fulfilled, web developers’ need is satisfied. Let’s call this as “Adobe’ s approach”, I will lay down the reasons in the second last paragraph.

If this Adobe’s approach is taken, does it bring any new security issue? No, none. Since any request made to http://2nd.com/ is what a.html needed and in a way prescribed by it: The responses from http://2nd.com/ is treated as data, either json objects or text, or html. a.html has control on the use of it, and any security check could be done before make use of it. If you think it brings any new security issue, please enlighten me!

If this Adobe’s approach is taken, does it solve some security issues? Yes, at first, the approach with script element or with jsonp can be avoided. As explained earlier, they are not secure since they are javascript code, and executed before being examined by its user – a.html. Furthermore, cross site attack can be avoided even an attacker from http://3rd.com/ (I do not like the term evil here either) can inject any strings (codes) into a.html. Since a.html did not tell browsers to deem http://3rd.com/ as same origin, so browser will not allow any request to be made to http://3rd.com/. By the way, with CORS, for the same situation, confidential data can be sent to http://3rd.com/receiver without any obstacle. Some people might not get this, so let me state it one more time: If the attacker from http://3rd.com/ has the ability to inject string into your bank http://yourBank.com/, then the attacker can send confidential data to http://3rd.com/receiver because http://3rd.com/ accepts access from http://yourBank.com/. If you think I am wrong, please enlighten me.

As for our original purpose, from web developers’ point of view, this Adobe’s approach is the best solution. It meets the cross site needs of web developers by supporting data access to other sites in a secure manner while not bringing any new security issue.

Be noted a.html has access to https://2nd.com/publicData does not implies it has access to https://2nd.com/secreteData as 2nd.com has authorization check procedure for its secreteData. Don’t tell me that manager at 2nd.com do not do authorization check for its protected data at server side, and solely rely browsers to do the authorization check. Stefan Zager is a nice reminder: at server side the white list approach should be taken, only resources in the white list can bypass some authorization check. Don’t tell me that the manager forgot to do so, and so browsers step forward and nicely take the responsibility for the protection of his confidential data. Don’t tell me you think the referrer based authorization check can be safely delegated from server side to browsers. With CORS, the referrer based authorization check reduced to domain based check. Any way, if you do, I would say that’s stupid.

All the respondents to my post murmured about cross site security issues. But no one really gets to the substantial point: the simple Adobe’s approach I proposed brings what extra security issues. I would say none. If you think there is any, please enlighten me!
Browser Sandbox Structure Does Not Entail CORS
If we look at the problem from browser designers’ point of view, as I promised before “I will talk about the securities of 2nd.com later”. The same site/origin policy is a derived term from the sandbox structure utilized in browsers to prevent different web applications from interfering each other. Each of different websites is put into different sandboxes. It is the sandbox structure that obstructs cross site data access.

There might be some websites, since they do not have any real confidential data to protect so they do not do authorization check at server side, instead they rely on browsers to do the domain based referrer authorization check (I have to admit that this sentence is logically flawed) or websites do have authorization check at server side, but still they want to utilize an extra layer of authorization, is there any way for resources to instruct browsers that the resources should only be put into some specific sandboxes? We can say to this aim, CORS is devised, clearly this is against the parsimony principle since domain base referrer authorization check if needed should be done at server side.
CORS Should Retire
Some people like Tab Atkins (post #0028) and Stefan Zager(post #0030) said it would be better that by default all resources should only be allowed to be put into the sandbox for its website, and should not be put into any other sandboxes unless stated otherwise. Let’s call this as “mandatory directive”. Tab Atkins acknowledged in post #0028 that “(As Jake has said, some resources that were linkable in the early web, like images, are stuck with #1. Changing them at this point would break too much of the web, but as a result we have regular security issues.)” I hope nobody misunderstood the last sentence which says if we impose the “mandatory directive”, then “too much of the web” is broken. In fact, not only “the early web”, many new websites do not conform to the mandatory directive by serving the header that Tab Atkins suggested and Florian Bosch is eager for(Access-Control-Allow-Origin: *). The language ability of Tab Atkins is the height that I am not able to achieve. When he said that, he gave people the impression that he is opposing the change in order not to “break too much of the web”, isn’t it? In fact, it is he who insisted “the mandatory directive” which “breaks too much of the web”. Did you misunderstood him as a result of his misleading sentence?

If the mandatory directive of CORS can be removed and add Adobe’s approach I proposed above, then we achieve the seamless extension of the same origin policy. It is OK to remove the mandatory directive as no confidential data on any website is dependent on it. By the way, when browsers found any resource cross site referred does not conform to the mandatory directive, browsers, if Tab Atkins likes, can make an extra request to notify the server such as http://2nd.com/CORSmandatoryViolation/theReferredResouce. Although this request will not have a response, when the manager of 2nd.com look at the log, they will notice it.

In a word, even to keep the CORS, there is no point to impose the mandatory directive. And I really hope the Adobe’s approach I proposed can be added to avoid “breaking too much of the early web” and the web that does not conform to CORS’ mandatory directive and to prevent cross site attacks even the attacker got the opportunities to inject strings!

Of course, you can always say: CORS – the domain based referrer authorization check at browser side – as an extra layer of security does no harm. I would say if you utilize CORS to do domain based referrer authorization checking done mainly by browsers, you know it is redundant even if you need it since you must have done so at server side, and if you rely on that, you are encouraged by those incompetent professionals to build a cranky security system! I would guess in reality when CORS is used, most of them is the header that Florian Bosch is eager for. Overall for parsimony reason, CORS should retire.

As Jake Archibald mentioned, semantics of Adobe “allowOrigin” in a.html(if) is to allow code from http://2nd.com/ to be executed in http://1st.com/a.html. In web, this is implicit as code can always be loaded and executed from anywhere. Given our original purpose on extending the same origin policy is for data access, so “allowOrigin” should have the semantics of instructing browsers to allow a.html to make request to http://2nd.com/ for data. This is the reason I think why we can call this approach as Adobe’s approach.

I feel the point I want to make is simple, but I found that people can not understand me. Is it because the words such as “stupid” drag people off the road to point I want to make? Originally, I thought as long as I raised the question, you smart people could right the wrong, and I do not have to spend time on this. Unfortunately, it did not go well, so I write this long post. If you find any mistake in this post, please let me know where I am wrong, but please do not utter/murmur some general statements while not getting to any substantial point.

20171013
Jack
