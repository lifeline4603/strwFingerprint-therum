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
strwFingerprint was a passion project I started when I left the cheating community. the whole idea behind the project was to take everything I learned from being in that community and then using it against them, which eventually turned into strwFingerprint. the philosophy behind it was that anticheats are far too cat and mouse, especially considering the fact that it's lua vs c++, so I just turned to the actual root of the problem. if the cheaters can't even join your server, then there's no need for an anticheat. initially strwFingerprint was relatively basic, as it checked the following things only:
- a cookie inside cl.db and \data\
- timestamps in \html\ (persistent)
- timestamps in \bin\ (persistent)

it was basic at the start, but eventually through brainstorming, I added the following methods of tracking:
- tracking \cstrike\, \tf\, etc. any source games that are mounted can be inspected and tracked with timestamps
- steam cloud (timestamps of demos)

steam cloud was really powerful because most people don't think of steam cloud as a detection vector. if they record a demo, that's synced up with the cloud and is a tracking vector across accounts.

the turning point for strwFingerprint was when I was watching instagram. I got one of those peter explains tech stuff AI voice videos (slop) and it was talking about **canvas fingerprinting,** where browsers can use pixels to track people across the web. now, me being the lunatic I am, I immediately jumped on my PC and researched about this, and what followed was hilarious.

## browser fingerprinting
websites use a clever and predatory trick called canvas fingerprinting! they essentially render a bunch of pixels on your screen, and every single GPU in the world computes it ever so slightly differently, and that allows predatory websites to create a hash of you. you've probably already been a victim of this, because a lot of websites you use on a daily basis use canvas fingerprinting to track you across the web. instagram, or more specifically [meta, is currently getting sued in germany](https://www.meta-klage.de/) for something very similar, as they process people's data specifically without the user's consent. it happens in the background silently.

I eventually figured out how to put this on garry's mod by using canvas, and then using fnv1a to turn it into a trackable hash. this essentially gave us a soft-HWID hash. it could have cross contamination, but ideally you'd use other browser fingerprinting techniques with it. I tested this a lot and got great results (gpu x gpu, os x os, device x device). there is a possibility that this has been patched, but it was working 8 months ago.

I essentially had a nuke on the alting scene. my plans went further. I was planning on combining these detected people into a database, and then having every server with strwFingerprint share a database of these alts. the servers can choose to ban or investigate them on join. servers could share information between each other on alt accounts easily like this. it was going to be insane. the issue however starts within the very first sentence of this section: it's clever and predatory. alt detection like this poses a huge issue, because at some point you're going to have to think about GDPR. GDPR has rarely ever been an issue for individual servers. yes, they have to abide by it, but even if you don't, the likelihood of you running into issues with the EU or proper authorities is relatively low unless you're a big brand (which for something like a TTT server is unlikely). for me it starts becoming an issue when I start having a shared database between a lot of servers, because that turns strwFingerprint, or more specifically Strawhat Fanclub into a big time data controller.

## what happened though?
I gave up! I got frustrated on the front end of things (ui code sucks on gmod fuck you rubat xoxo) so I didn't want to bother and moved on. the detections were working. everything was. I just couldn't design a proper UI. the backend was also kinda fucked since I had issues with my VPS, but I digress. the legal issues that could follow (like having to follow up with GDPR requests) are just not worth my time, which is why I scaled back initially and scrapped the database idea temporarily.

## how it worked

this is the original prototype, trimmed down. it's old and shit but it worked. the full thing essentially consisted of:

1. open a browser (DHTML) inside gmod
2. draw text onto a canvas and read the pixels back
3. hash the pixels with fnv1a
4. send the hash from javascript back to lua
5. send it to the server and log it next to the player's steamid

gmod ships with CEF (chromium), so any DHTML panel is basically a full browser (god I love browsers in my games). AddFunction opens a lua function to javascript as gmod.receiveHash(...), which is how the page can talk to the game
```lua
local html = vgui.Create("DHTML", frame) // frame == parent panel btw
html:Dock(FILL)

html:AddFunction("gmod", "receiveHash", function(hash)
    if not hash or hash == "" then return end
    net.Start("strwfp_hash")
    net.WriteString(hash)
    net.SendToServer()
end)

html:SetHTML(htmlContent)
```

the way you can actually get a fingerprint is by painting a canvas white and rendering some text on it. text rendering (anti-aliasing, font smoothing, subpixel stuff, all that crazy bullshit) comes out slightly different depending on your gpu, drivers and os, so the pixels you read back are slightly different from machine to machine

my prototype also bundled some other browser data into the hash.
```js
function buildSampleString(ctx, c) {
    ctx.fillStyle = "#fff";
    ctx.fillRect(0, 0, c.width, c.height);
    ctx.fillStyle = "#000";
    ctx.font = "16px Arial";
    ctx.textBaseline = "top";

    var info = [
        navigator.userAgent || "",
        screen.width + "x" + screen.height,
        screen.colorDepth || "",
        navigator.language || "",
        new Date().getTimezoneOffset(),
        navigator.platform || "",
        navigator.hardwareConcurrency || ""
    ];

    for (var i = 0; i < info.length; i++) {
        ctx.fillText(info[i], 4, 4 + i * 12);
    }

    var data = ctx.getImageData(0, 0, c.width, c.height).data;

    var s = "";
    for (var k = 0; k < data.length; k += 64) {
        s += String.fromCharCode(data[k] & 0xff);
    }
    return s + "|" + info.join("|");
}
```

now we need to hash it! the string we get back is long and absurd, and we don't need that, so instead we squish it into a short, beautiful and pretty hash with fnv1a because it's the most simple and at it's time most convenient.
```js
function fnv1a(str) {
    var hash = 0x811c9dc5;
    for (var i = 0; i < str.length; i++) {
        hash ^= str.charCodeAt(i);
        hash = (hash * 0x01000193) >>> 0;
    }
    return hash.toString(16);
}
```

the html tries the gmod.receiveHash path first (see earlier snippets). if that's not there or fails for whatever reason, it falls back to navigating to a fake gmod:// url, and lua catches the url change and pulls the hash out of it.
```js
function sendToGmod(hash) {
    if (window.gmod && window.gmod.receiveHash) {
        window.gmod.receiveHash(hash);
        return;
    }
    window.location.href = "gmod://fingerprint:" + hash; // epic fallback :3
}
```
```lua
function html:OnURLChanged(url)
    local prefix = "gmod://fingerprint:"
    if url and string.sub(url, 1, #prefix) == prefix then
        local hash = string.sub(url, #prefix + 1)
        net.Start("strwfp_hash")
        net.WriteString(hash)
        net.SendToServer()
    end
end
```

my cool prototype just printed the hash and the player. the plan from here was to store it, check new joins against it, and eventually share that between servers (which is where GDPR comes in!! see above).
```lua
util.AddNetworkString("strwfp_hash")

net.Receive("strwfp_hash", function(_, ply)
    local hash = net.ReadString()
    print(string.format("%s (%s) -> %s", ply:Nick(), ply:SteamID(), hash))
end)
```

this was a small scale test, so ideally you'd want to sample more pixels and channels, and be more specific with your targets (like don't track resolution in the hash). I also never accounted for exploiters bypassing this (never trust the client guys #1 mistake), so that's another vector I have genuinely 0 idea how to avoid. another funny epic thing rubat could do that would fuck with all the hashes is update CEF, which hasn't been updated in YEARS, even after requests. knowing rubat, the right update at the wrong time could make all the difference in the world. so wake up, mr freeman.
