Background

A honeypot is a decoy application, server, or other networked resource that intentionally exposes insecure features which, when exploited by an attacker, will reveal information about the methods, tools, and possibly even the identity of that attacker. Honeypots are commonly used by security researchers to understand the threat landscape facing developers and system administrators, collecting data that might include:


    Information about sources of malicious network traffic such as IP addresses, geographic origin, targeted ports, etc.
    Information used to harden resources against email spammers
    Malware samples
    DB vulnerabilities such as SQLI techniques

There are two broad categories of honeypots:

    Low-interaction honeypots provide simulations of target resources, typically using emulation or virtualization, to reduce resource consumption, simplify configuration and deployment, and provide solid containment features
    High-interaction honeypots expose non-simulated target resources in a way that more closely imitates a production environment to attract more sophisticated attackers and understand more complicated exploitation routes

For example, a low-interaction honeypot might emulate a server capable of accepting SSH connections through a combination of exposed ports and decoy responses, whereas the high-interaction version would feature an actual SSH server possibly misconfigured in some way that makes it vulnerable. In the low-interaction example, attempted exploitation would quickly lead to a dead end for the attacker, perhaps revealing only an IP address and a few attempted commands to the honeypot's maintainer. In the high-interaction example, the attacker would potentially be able to compromise the server, wasting more time and giving away more information about his or her goals
