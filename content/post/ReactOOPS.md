+++
date = '2026-01-26T19:50:27-08:00'
title = 'HTB: ReactOOPS'

image = "/post/images/ReactOOPS.jpg"
tags = ["CTF", "HTB", "CVE"]
categories = ["HTB"]
+++

# HTB: ReactOOPS
- Difficulty: Very Easy
- Category: Web
- Solved: 1/26/2026

## Challenge Description
> NexusAI's polished assistant interface promises adaptive learning and seamless interaction. But beneath its reactive front end, subtle glitches hint that user input may be shaping the system in unexpected ways. Explore the platform, trace the echoes in its reactive layer, and uncover the hidden flaw buried behind the UI.

![Alt text](/post/images/ReactOOPS_webpage.png)

## Recon
I started by digging around the codebase which led to a few key findings:
- `package.json` - Revealed the package name `react2shell` along with the frameworks versions
    - Next.js `16.0.6`
    - React `^19`
- The `Dockerfile` showed a flag located at `/app/flag.txt`

Since the challenge description hinted at user input but there were no input forms on the page this seemed to indicate that the vulnerability was in the framework no the application code. 

## Vulnerability Identification
A quick google search of "react2shell" (just using my resources not cheating ... right?) pointed towards a a known vulnerability [CVE-2025-55182](https://nvd.nist.gov/vuln/detail/CVE-2025-55182). 

## How the Vulnerability Works
The vulnerability exists in React's Flight protocol deserializer, specifically in the `getOutlinedModel()` function in `ReactFlightReplyServer.js`. A missing `hasOwnProperty` check during property traversal allows prototype chain pollution.
```javascript
// Vulnerable code
for (let i = 0; i < path.length; i++) {
    value = value[path[i]];  // No hasOwnProperty check!
}
```

The attack chain then looks something like this:
1. Send a reference like `$1:__proto__:then` in a POST request
2. This traverses the prototype chain to reach `Chunk.prototype.then`
3. React interprets the object as a Promise (due to the `.then` property)
4. When React awaits the "Promise", it executes attacker-controlled code
5. The code runs in Node.js context with server privileges

## Exploitation
I used an exploit script that I found on [github](https://github.com/freeqaz/react2shell/tree/master).

```bash
./exploit-redirect.sh http://<TARGET>:<PORT>/ "cat /app/flag.txt"
```

To get this result

```bash
[+] HTTP 303 - Redirect exfil successful
[+] Command output:
----------------------------------------
HTB{<flag>}
----------------------------------------
```

## Takeaways
1. You are only as strong as your weakest framework. In this example the application code was sound but an exploit in the framework left is vulnerable. 
2. Sometimes the answer is in the package.json! It would never be this easy in the real world but for these little puzzles combing through the files can hold all kinds of little (or big) hints. 
3. The initial exploit triggered RCE but didn't return output. Using Next.js's redirect mechanism allowed us to capture command output via response headers.

## References
- [https://nvd.nist.gov/vuln/detail/CVE-2025-55182](https://nvd.nist.gov/vuln/detail/CVE-2025-55182)
- [https://www.trendmicro.com/en_us/research/25/l/CVE-2025-55182-analysis-poc-itw.html](https://www.trendmicro.com/en_us/research/25/l/CVE-2025-55182-analysis-poc-itw.html)
- [https://www.wiz.io/blog/critical-vulnerability-in-react-cve-2025-55182](https://www.wiz.io/blog/critical-vulnerability-in-react-cve-2025-55182)
- [https://react.dev/blog/2025/12/03/critical-security-vulnerability-in-react-server-components](https://react.dev/blog/2025/12/03/critical-security-vulnerability-in-react-server-components)

