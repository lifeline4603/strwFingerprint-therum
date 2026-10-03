<div align="center">
    <img src="memo.png"
        title="strwFingerprint - Therum" alt="logo" width="75"/>
    <h1>strwFingerprint - Therum</h1>
    <p>
        an anti-alting system based on browser fingerprinting, taking advantage
        <br>
        of CEF, a shared database and beyond.
    </p>
</div>

## what is strwFingerprint?
strwFingerprint was a passion project I started when I left the cheating community. the whole idea behind the project was to take everything I learned from being in that community and then using it against them, which eventually turned into strwFingerprint. initially strwFingerprint was relatively basic, as it checked the following things only:
- a cookie inside cl.db and \data\
- timestamps in \html\ (persistent)
- timestamps in \bin\ (persistent)

it was basic at the start, but eventually through brainstorming, I added the following methods of tracking:
- tracking \cstrike\, \tf\, etc. any source games that are mounted can be viewed inside of
- steam cloud (timestamps of demos)

steam cloud was really powerful because most people don't think of steam cloud as a detection vector. if they record a demo, that's synced up with the cloud and is a tracking vector across accounts.

the turning point for strwFingerprint was when I was watching instagram. I got one of those peter explains tech stuff AI voice videos (slop) and it was talking about **canvas fingerprinting,** where browsers can use pixels to track people across the web. now, me being the lunatic I am, I immediately jumped on my PC and researched about this, and what followed was hilarious.

## browser fingerprinting
websites use a clever and predatory trick called canvas fingerprinting! they essentially render a bunch of pixels on your screen, and every single GPU in the world computes it ever so slightly differently, and that allows predatory websites to create a hash of you. you've probably already been a victim of this, because a lot of websites you use on the daily basis use canvas fingerprinting to track you across the web. instagram, or more specifically meta, is currently getting sued in germany exactly for this, as they process this data specifically without the user's consent. it happens in the background silently.

I eventually figured out how to put this on garry's mod by using canvas, and then using fnv1a to turn it into a trackable hash. this essentially guaranteed a HWID hash. I tested this a lot and got great results. there is a possibility that this has been patched, but it was working 8 months ago.

I essentially had a nuke on the alting scene. my ideas were further. I was planning on combining these detected people into a database, and then having every server with strwFingerprint share a database of these alts. the servers can choose to ban or investigate them on join. servers could share information between each other on alt accounts easily like this. it was going to be insane.

## what happened though?
I gave up! I got frustrated on the front end of things (ui code sucks on gmod fuck you rubat xoxo) so I didn't want to bother and moved on. the detections were working. everything was. I just couldn't design a proper UI. the backend was also kinda fucked since I had issues with my VPS, but I digress.

more coming soon like snippets and shit.
