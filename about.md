---
title: About
permalink: /about/
layout: resume
comments: false
description: >-
  Benham Haynes (xPOPE): systems administrator, founder of UnChat, and
  student leader building and breaking things in cybersecurity.
---

<article class="resume" itemscope itemtype="https://schema.org/Person">
  <header class="resume-head">
    <h1 class="resume-name" itemprop="name">Benham Haynes <span class="resume-handle">(xPOPE)</span></h1>
    <p class="resume-role">Systems Administrator &nbsp;|&nbsp; Founder, UnChat &nbsp;|&nbsp; Student Leader</p>
    <ul class="resume-contact">
      <li><a href="https://www.linkedin.com/in/benhamhaynes" target="_blank" rel="noopener">LinkedIn</a></li>
      <li><a href="https://github.com/xPop3" target="_blank" rel="noopener">GitHub</a></li>
      <li><a href="https://unchat.app" target="_blank" rel="noopener">UnChat</a></li>
      <li><a href="https://scbbg.org" target="_blank" rel="noopener">SCBBG</a></li>
    </ul>
    <p class="resume-certs">CompTIA Security+ &nbsp;&middot;&nbsp; Certified Ethical Hacker (CEH) &nbsp;&middot;&nbsp; Certified Digital Forensics Examiner (CDFE)</p>
    <p class="resume-download">
      <a href="{{ "/resume/" | relative_url }}">View Resume</a>
    </p>
  </header>

  <section class="resume-section">
    <h2>About Me</h2>
    <p>I'm Pope (Benham Haynes), a cybersecurity and IT professional who builds things, breaks things, and then writes about how both went. Right now I work as a Systems Administrator at Get IT Right Solutions (GripD), a managed service provider, while running my own SaaS product, leading a student security club, and getting ready to start a B.S. in Cybersecurity and Information Assurance at WGU. My long-term goal is a career that spans both sides of security: offensive work that finds the gaps, and defensive work that closes them.</p>
  </section>

  <section class="resume-section">
    <h2>Where I Started</h2>
    <p>I didn't take the traditional path into tech. Before I ever touched a firewall, I spent more than four years as a professional hairstylist, and I still work at Arca Salon in Costa Mesa. That career taught me things no certification covers: how to listen to what someone actually wants, how to explain something technical to someone who doesn't care about the technical part, how to stay calm when a client is unhappy, and how to do detailed, precise work while carrying a conversation. Those habits turned out to be exactly what IT support and security work demand. When users call an MSP, they're usually stressed, and the job is half fixing the problem and half making them feel like it's handled.</p>
    <p>Somewhere along the way, curiosity about how networks and systems worked turned into a serious pursuit. I went back to school, committed fully, and started building a skill set from the ground up.</p>
  </section>

  <section class="resume-section">
    <h2>Education</h2>
    <p>In October 2026 I completed my A.S. in Network Administration at Saddleback College, graduating Summa Cum Laude with a 4.0 GPA, along with a Certificate of Achievement in Security Analysis. My coursework covered networking fundamentals at the Network+ level (DNS, VLANs, ACLs, subnetting, and firewall configuration), C++ programming, and applied statistics using Python and SciPy for hypothesis testing, regression, and ANOVA.</p>
    <p>Outside the classroom, I served as a class liaison for the Saddleback Computer Science Department, acting as a bridge between students and faculty. My next step is transferring to Western Governors University for a B.S. in Cybersecurity and Information Assurance.</p>
  </section>

  <section class="resume-section">
    <h2>Certifications</h2>
    <ul>
      <li>CompTIA Security+</li>
      <li>Certified Ethical Hacker (CEH)</li>
      <li>Certified Digital Forensics Examiner (CDFE)</li>
    </ul>
    <p class="resume-note">Together these cover the defensive foundation, the attacker's mindset, and the investigative side of what happens after an incident.</p>
  </section>

  <section class="resume-section">
    <h2>Professional Experience</h2>
    <div class="resume-entry">
      <div class="resume-entry-head">
        <h3>Systems Administrator: GripD dba Get IT Right Solutions (MSP)</h3>
      </div>
      <p>At GripD, I handle the day-to-day reality of supporting multiple client environments.</p>
      <ul>
        <li><strong>Firewall administration:</strong> configuring and maintaining SonicWall appliances for client networks.</li>
        <li><strong>Microsoft 365 administration:</strong> Exchange Online management (including dynamic distribution groups), tenant configuration, and Microsoft 365 Copilot policy setup and optimization.</li>
        <li><strong>Network hardening:</strong> making IPv6 security decisions for client networks so new protocols don't quietly open new attack surface.</li>
        <li><strong>Security awareness:</strong> building and running Gophish phishing simulation campaigns to train users to spot real attacks.</li>
        <li><strong>Vendor and remote access:</strong> setting up secure remote access for third-party vendors.</li>
        <li><strong>Guest network engineering:</strong> designing and deploying a branded captive portal for a hospitality client using IronWifi and Ruckus hardware, including SMS-based authentication, custom splash pages, and terms and privacy documentation. Getting it working inside the restrictive captive-portal mini browser meant tracing a conflict between two layers of MAC/IP encryption, which is the kind of problem I genuinely enjoy chasing down.</li>
      </ul>
      <p>I've also been building an AI-assisted help desk agent for MSP workflows, with a strong emphasis on privacy. The core of that project is a privacy gateway I designed: real client identifiers stay local, the external AI model only ever sees reversible, scoped pseudonyms, secrets are irreversibly redacted, and the whole system fails closed when anything looks uncertain. Every request is audit-logged without storing the mappings themselves. It's a practical answer to a question every MSP is asking right now: how do you use AI without handing your clients' data to a third party?</p>
    </div>
  </section>

  <section class="resume-section">
    <h2>Founder: UnChat</h2>
    <p>I'm the founder and sole developer of <a href="https://unchat.app" target="_blank" rel="noopener">UnChat</a>, an AI text humanization SaaS. I built it end to end on Next.js, Supabase, Stripe, and the Anthropic API, took it from idea to live deployment, and continue iterating on the core humanization logic and on how to evaluate output quality honestly. Running UnChat has taught me the business side of tech: pricing, payments, product decisions, and shipping something real instead of just a demo.</p>
  </section>

  <section class="resume-section">
    <h2>Competitions and Offensive Security</h2>
    <p>Hands-on competition is where I sharpen my skills under pressure. I've competed in:</p>
    <ul>
      <li>SoCal Cyber Cup qualifier rounds</li>
      <li>CIRCUS, where I captained the Forensics and Incident Response team</li>
      <li>HackCCDC / CCDC, defending live infrastructure against active red teams</li>
      <li>Ongoing challenges on Hack The Box, TryHackMe, and Cyber Skyline</li>
    </ul>
    <p>I document machine walkthroughs and lab work as part of my study process, because if I can't explain how I got in, I don't really understand it yet.</p>
  </section>

  <section class="resume-section">
    <h2>Leadership: SCBBG</h2>
    <p>I lead the <a href="https://scbbg.org" target="_blank" rel="noopener">Saddleback Cyber Builders and Breakers Group (SCBBG)</a>, a student cybersecurity club I built out and continue to develop. The name says it all: we build systems and we break them. I create club materials, run hands-on projects, and try to give students the lab experience I wish I'd had when I started. One of our club builds is <a href="{{ "/shadowapple-wifi-pineapple/" | relative_url }}">ShadowApple</a>, a DIY Wi-Fi Pineapple I assembled on a GL.iNet AR300M running OpenWrt with PineAP, mounted in a rugged hard case with external antennas for wireless security demonstrations.</p>
  </section>

  <section class="resume-section">
    <h2>The Homelab</h2>
    <p>Most of what I know, I learned by building it at home first. My lab runs on a Proxmox host (Dell Precision T7910) alongside a Dell PowerEdge R520 running Hyper-V, and it's built like a small business network:</p>
    <ul>
      <li>SonicWall TZ500 as the edge firewall (an upgrade from a consumer router that gained roughly 250 Mbps of throughput)</li>
      <li>UniFi wireless with a self-hosted controller</li>
      <li>AdGuard Home for network-wide DNS filtering</li>
      <li>A dedicated WireGuard VPN gateway container routing traffic through Mullvad, with DNS pinned to my own resolver</li>
      <li>Tailscale for secure remote access across devices</li>
      <li>A fully isolated vulnerable lab network with no internet access, where Kali and Parrot OS attack boxes are dual-homed against targets like Metasploitable 2</li>
      <li>Local LLM inference served over the LAN from my workstation, so I can experiment with AI tooling without sending data off-network</li>
    </ul>
    <p>Read the <a href="{{ "/xpope-lab-homelab/" | relative_url }}">full homelab writeup</a>. I've also built a MediCat/Ventoy field toolkit for repair and recovery work, jailbroken an iOS device for mobile security testing with tools like Frida and nmap, and built several automation projects, including a Debian-based trading bot and an Instagram content pipeline with a Discord approval workflow.</p>
  </section>

  <section class="resume-section">
    <h2>Freelance QA</h2>
    <p>I've done freelance quality assurance work for Bit Developers on PrayerConnect, a web-based prayer management application, running multiple rounds of structured testing, writing reproducible bug reports, and building testing checklists. QA trained my eye for edge cases, which carries straight over into security work: the bugs that matter are usually the ones nobody thought to test.</p>
  </section>

  <section class="resume-section">
    <h2>What Drives Me</h2>
    <p>What ties all of this together is a simple habit: I don't trust a system until I understand how it fails. Whether that's a firewall rule, a captive portal, an AI pipeline handling client data, or a vulnerable VM in my lab, I want to know where it breaks and why. I bring the same people skills from a decade-plus of client-facing work to every ticket, every team, and every competition.</p>
    <p>I'm always open to connecting with people in security, IT, and the builder community. If you're working on something interesting, or want to talk shop about labs, CTFs, or privacy-first AI, <a href="https://www.linkedin.com/in/benhamhaynes" target="_blank" rel="noopener">reach out</a>.</p>
  </section>
</article>
