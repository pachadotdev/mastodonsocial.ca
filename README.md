# mastodon_setup

Dear people,

With Twitter uncertainty, I decided to create my own Mastodon instance.

Mastodonsocial.ca is a public online space oriented to different universities alumni, faculty, staff, community, and professional students looking to grow their social circles around serious and light-hearted academic discussion and other related fields.

With an account on mastodonsocial.ca you'll be able to follow people on any Mastodon server and beyond.

I kindly suggest an optional donation on buymeacoffee.com/pacha. Here you'll find ethical design: encrypted connection, no trackers, no 3rd party requests, no ads.

Best regards,
Pacha

# Annotations on how to update the server

## 2022-11-16

I started with Mastodon 3.5.3 provided in the 1-click Mastodon image (https://marketplace.digitalocean.com/apps/mastodon). This image does **not** use Docker. The same day I tried to update to v4.0.2 following the instructions from https://docs.joinmastodon.org/admin/upgrading/, it failed. Then I tried https://fedi.dev/gytis/update-mastodon-server-instance and verified all the minimal versions, it still didn't work. I opted for rebuilding the image, a DB restoration didn't work, so I restored a 2nd time and started from zero.

Sudomarch recommends this based on his own experience "Reinstall 3.5.x, restore your db, and then do the upgrade to 4.0.0, and THEN 4.0.2. That's what worked for me" (https://www.reddit.com/r/Mastodon/comments/yxjvea/no_web_view_after_updating_to_402/). I'll postpone updates until it's necessary to keep connected to mastodon.

## 2022-11-21

To be cautious, I created a snapshot of the droplet before, to avoid losing the existing accounts. All the images and assets are in S3.

I found https://github.com/mastodon/mastodon/releases/tag/v4.0.0 but `bundle install` doesn't work, so I completed the next steps based on the previous link and https://fedi.dev/gytis/update-mastodon-server-instance.

Update 3.5.3 to 4.0.0

```bash
# After login as root from DO web console
systemctl stop mastodon*
su mastodon
cd /home/mastodon/live
git fetch && git checkout v4.0.0
nano .ruby-version
bundle install && yarn install
SKIP_POST_DEPLOYMENT_MIGRATIONS=true RAILS_ENV=production bundle exec rails db:migrate
RAILS_ENV=production bundle exec rails assets:precompile
exit
systemctl start mastodon-sidekiq.service 
systemctl start mastodon-streaming.service 
systemctl start mastodon-web.service
su mastodon
cd /home/mastodon/live/
RAILS_ENV=production bundle exec rails db:migrate
exit
```

# 2022-11-23

To be cautious, I created a snapshot of the droplet before, to avoid losing the existing accounts. All the images and assets are in S3.

I realized I needed a swap file or the compilation process fails. Probably we need less than 8GB for the swap.

Update 4.0.0 to 4.0.2

```bash
# After login as root from DO web console
fallocate -l 8G /swapfile
mkswap /swapfile
swapon /swapfile
systemctl stop mastodon*
su mastodon
cd /home/mastodon/live/
git fetch && git checkout v4.0.2
bundle install
RAILS_ENV=production bundle exec rails assets:precompile
exit
systemctl start mastodon-sidekiq.service 
systemctl start mastodon-streaming.service 
systemctl start mastodon-web.service
swapoff /swapfile
rm -f /swapfile
```
