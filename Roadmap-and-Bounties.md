Roadmap for 2020
========
- Introduce a new streaming backend architecture. See progress https://github.com/flashmob/go-guerrilla/pull/135
- Make the logrus dependency softer and have the ability to bring-your-own logger (Several proposals and PoC have been added already)
- improve some of the logged messages with more structure for easier parsing
- Support internationalized email addresses, and better 8bitmime support
- (Maybe?) Steam based DKIM verification
- Better tools for parsing of MIME, transforming encoding & storage deduplication


Bounties
===========

To encourage more development, we are now offering bounties 
funded from our ETH fund.

So far we have the following bounties that are still open:
(Updated 12 Mar 2017)

| What   | Bounty | Status | Description |
|--------|--------|--------|-------------|
|Let's encrypt TLS certificates!|0.5 ETH| Discussion | Closed. Take a look at [issue #29](https://github.com/flashmob/go-guerrilla/issues/29)
|Automated Testing| 0.1 ETH | Ongoing | Already paid some bounties, more welcome. Award judged based on a satisfactory increase in coverage. Please open an issue before to discuss scope.                                     
|Profiling| 0.25 ETH | Open | Simulate a configurable number of simultaneous clients  (eg 5000) which send commands at random speeds with messages of various lengths. Some connections to use TLS. Some connections may produce errors, eg. disconnect randomly after a few commands, issue unexpected input or timeout. Provide a report of all the bottlenecks. (Flame graph maybe? https://github.com/uber/go-torch Please open an issue before to discuss scope)
|Code Review | 0.25 ETH | Ongoing | Looking for someone to do a code review & possibly fix any tidbits, they find, or suggestions for doing things better. (Already one bounty of 0.25 paid, however, more is always welcome)

Recently completed:


| What   | Bounty | Status | Description |
|--------|--------|--------|-------------|
|Analytics dashboard| 1 BTC | Completed | See branch https://github.com/flashmob/go-guerrilla/tree/dashboard
|Fuzz Testing | 0.25 BTC | Completed | See Result https://github.com/flashmob/go-guerrilla/wiki/Fuzz-testing

Ready to roll up your sleeves and have a go?

Please open an issue for more clarification / details on Github.

Also, welcome your suggestions for adding things to this Roadmap - please open an issue.

