# Changelog

### v0.2.0

*Released: January 25th 2018*

- Remove port 80 / 443 from being open by default
- Allow all ICMP traffic (you can ping your servers now)
- Change systemd service to start before `network-pre.target` instead of `network.target`
- Add proper tests and support for Ubuntu 16, Debian Stretch and Debian Jessie
- Update format and style consistencies

### v0.1.2

*Released: October 9th 2016*

- Ensure `iptables` is installed
- Test against Ubuntu 16.04 LTS and Debian Jessie on Travis-CI

### v0.1.1

*Released: October 7th 2016*

- Fix tests to reference the correct role

### v0.1.0

*Released: October 7th 2016*

- Initial release
