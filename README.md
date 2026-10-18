# install wordpress on vultr: the One-Click app route, which plan to pick, and the SSL and domain steps that trip people up

The quickest way to install WordPress on Vultr is the One-Click WordPress app. You pick a server location, choose a plan, click Deploy Now, and the full stack — Nginx, PHP, MySQL/MariaDB, and WordPress itself — comes up in about a minute. Everything after that is setup work: an SSL certificate, the WordPress installation wizard, and pointing the site at your domain name.

This guide covers the whole run, from signup to a logged-in dashboard, with extra attention on the parts short tutorials tend to skip: the login popup that looks like something broke, the certbot command that keeps browsers from flagging your site, and the one WordPress setting you should change before you write a single post.

> **Short on time?** Deploy the One-Click WordPress app on the **$6/month High Frequency plan** (1 vCPU, 1 GB RAM, 32 GB NVMe). Vultr bills hourly, so a test server costs a few cents a day, and you can destroy it whenever you like. [👉 Deploy the One-Click WordPress app on Vultr](https://www.vultr.com/marketplace/apps/wordpress/?ref=7268283)

## Three ways to get WordPress onto a Vultr server

There are three realistic routes, and they differ mostly in how much server administration you want to own.

**The One-Click Marketplace app** ships a prebuilt image with Nginx, PHP, MySQL/MariaDB, WordPress, phpMyAdmin, Cockpit, Wordfence, and the SMTP Mailer plugin already configured. Deployment takes about a minute, and the remaining work happens in a browser-based wizard. This is the route most first-timers should take, and the rest of this guide follows it.

**A manual install on a clean OS** means spinning up Ubuntu or Debian, installing the LEMP or LAMP stack yourself, creating the database, downloading WordPress from wordpress.org, and wiring up wp-config.php plus your Nginx vhosts. More work, more control, and a decent way to learn how the pieces fit. Budget an evening instead of ten minutes.

**A third-party control panel** sits on top of Vultr and handles server provisioning for you. SpinupWP, for example, connects to your Vultr account through its API and provisions an Nginx/PHP-FPM/Redis server in about ten minutes, with a 7-day free trial. That route makes sense when you're juggling several client sites, though it adds a monthly fee on top of the server itself.

One thing to ignore: old tutorials that paste a "startup script" into the Vultr panel to auto-install Apache, MySQL, and PHP. The version still floating around was written for CentOS 6 back in 2014 and isn't guaranteed to work on anything newer. WordPress on Vultr has moved on.

## Which Vultr plan to pick for WordPress

For a fresh blog or small business site, the $6/month High Frequency plan is the sensible default: 1 vCPU, 1 GB RAM, and 32 GB of NVMe storage on 3GHz+ cores that chew through PHP requests faster than the regular tier. On a strict budget, the $5/month Regular Performance plan (1 vCPU, 1 GB RAM, 25 GB SSD) runs WordPress fine — you'll mostly notice the difference under load. If the site needs to carry real traffic or a WooCommerce store, plan for 2 GB of RAM or more; that's the standard production recommendation for WordPress servers, and Vultr lets you resize later, though the server goes offline briefly while it resizes.

Two billing details worth knowing before you pick. First, Vultr bills by the hour, so a $6 server costs pennies while you test and nothing once you destroy it. Second, there's no renewal-price jump to brace for: the price you see is the price you keep paying, which is more than budget shared hosting can usually claim. Automatic server backups are available as a paid add-on if you want them.

Here are the starting points side by side:

| Plan | Core specs | Starting price | Best for | Deploy link |
| --- | --- | --- | --- | --- |
| Cloud Compute – Regular Performance | 1 vCPU, 1 GB RAM, 25 GB SSD | $5/month (billed hourly) | Tight budgets, low-traffic blogs | [ Deploy on this plan](https://www.vultr.com/pricing/?ref=7268283) |
| Cloud Compute – High Frequency | 1 vCPU, 1 GB RAM, 32 GB NVMe | $6/month (billed hourly) | Default pick for new WordPress sites | [ Deploy on this plan](https://www.vultr.com/products/high-frequency-compute/?ref=7268283) |
| Larger Cloud Compute sizes | 2 GB RAM and up, more CPU and storage | Scales with size | WooCommerce, busier sites | [ See all current plans](https://www.vultr.com/pricing/?ref=7268283) |

You can [👉 check Vultr's current plan pricing](https://www.vultr.com/pricing/?ref=7268283) for the full catalog — there are larger tiers beyond these starters, and the prices are listed flat rather than as teaser rates.

## Step 1: Deploy the One-Click WordPress app

Deployment is mostly clicking through five choices in order:

1. Create a Vultr account and add a payment method. Billing is hourly, so nothing gets charged until a server actually runs.
2. In the control panel, go to Products → Deploy New Server, switch the Choose Image tab to Marketplace, and search for "WordPress."
3. Pick the region closest to your audience. Vultr runs 32 data center regions across six continents, so a reader in Mumbai gets a Mumbai server instead of one an ocean away.
4. Add an SSH key if you have one; it beats password logins for the terminal work later.
5. Click Deploy Now. The server is live in about a minute.

Before you close the panel, open the server's information page and note the credentials listed there. You'll need the root login for SSH in the SSL step, and the security user for the WordPress admin popup in the next few minutes.

[👉 Open the one-click WordPress deploy page](https://www.vultr.com/marketplace/apps/wordpress/?ref=7268283) if you'd rather skip the Marketplace hunt and go straight to the app.

## Step 2: Install a free SSL certificate (or skip it, knowingly)

You can run WordPress without SSL and just use the server's IP address, but you'll be clicking through certificate warnings constantly, and any site you actually plan to publish deserves better. The recommended path costs nothing: register a domain, point a DNS record at your server's IP, and install a free Let's Encrypt certificate with certbot, which comes preinstalled on the One-Click image.

1. Register a domain and create a DNS record for the server's IP, through your DNS provider or Vultr's own DNS.
2. Wait for DNS propagation. It's usually quick but can take up to 48 hours — confirm the domain resolves before moving on.
3. Connect via SSH using the root login.
4. Run certbot, listing both the www version and the bare domain:


certbot --nginx --redirect -d www.example.com -d example.com -m admin@example.com --agree-tos --no-eff-email


That single command issues one certificate covering both names and sets up the redirect from HTTP. Certbot reports success when it finishes.

If you skip this for now, know the catch: WordPress stores complete URLs in its database, so adding HTTPS months later can leave older content pointing at unencrypted http:// links. There are plugins that force everything to HTTPS and clean that up, but doing SSL first avoids the whole problem.

## Step 3: The wp-admin login popup is not an error

The first time you visit yoursite.com/wp-admin, your browser throws a "Sign in" dialog asking for credentials. This is the step that confuses people on nearly every first deploy. It's a deliberate security layer Vultr placed in front of WordPress while it's still unconfigured — enter the security user credentials from your server's information page and you're through.

Two related things you might run into:

- A privacy or certificate warning if you haven't installed SSL yet. The server uses a self-signed certificate by default; it's safe to proceed past it to finish setup.
- Reaching the admin area over plain HTTP with your server IP also works, but it's insecure and shouldn't become a habit.

Once WordPress is installed, you can disable the popup entirely; Vultr's documentation has the directions.

## Step 4: Run the WordPress wizard, then set your domain

The wizard itself is the easy part. Select your language and continue. Enter a site title, username, and email address. The installer suggests a strong password — save it or type your own — then click Install WordPress. When it finishes, you land in the standard WordPress dashboard.

Then do one thing before you publish anything: go to **Settings → General**, set both the WordPress Address (URL) and Site Address (URL) to your actual domain, and save.

This matters more than it looks. WordPress stores complete URLs in the database, so content you create while the site is still reachable at the raw IP will carry links pointing at that IP. Change the URLs afterward and some pages end up broken. The repair is a database search-and-replace plugin (after a backup), but spending thirty seconds on the URL settings first makes that unnecessary.

## Three things Vultr already installed for you

The One-Click image bundles a few extras, and two of them solve problems you'd otherwise hit within your first month.

**SMTP Mailer** — reliable email is a chronic weak spot for WordPress sites. Password resets, comment notifications, and contact form messages all depend on outbound mail actually arriving. The plugin comes preinstalled: activate it under Plugins → Installed Plugins, then configure it with your SMTP server's details.

**Wordfence** — an endpoint firewall and malware scanner for WordPress. It's preinstalled but inactive; turning it on gives you security logging and scanning from day one, which is exactly when a fresh site needs it.

**Cockpit** — a browser-based server control panel on port 9080, handy for poking at the server without SSH. Log in with the credentials from your server's information page. One gotcha: if you use the Vultr Firewall, you must explicitly allow port 9080 or the panel simply won't load.

## Frequently asked questions

**Can I install WordPress on Vultr without a domain name?**
Yes. Use the server's IP address in place of the domain, click through the certificate warning, and complete the whole setup. Just set the two URLs in Settings → General before creating content, so you're not cleaning up broken links later.

**How much does it cost to try this?**
Practically nothing. Vultr bills hourly, so a $5–6/month plan runs a few cents per day, and destroying the server stops the meter. New accounts also get periodic free-credit promotions — the amount varies with the current offer, so check what's listed when you sign up.

**Is that browser login popup on wp-admin normal?**
Yes. It's Vultr's security layer, covered in step 3 above, and it goes away once you install WordPress and disable it.

**What if I want WooCommerce?**
Vultr offers a separate One-Click WooCommerce app, and the setup steps are the same as the ones in this guide. Installing the WooCommerce plugin yourself on a standard WordPress install works too.

## So which setup should you go with?

Match it to what you're actually building:

- **Testing the waters or running a personal blog:** the $6 High Frequency plan (or $5 Regular if every dollar counts), the One-Click app, and the free Let's Encrypt cert. Total setup is well under an hour, and hourly billing means an abandoned experiment costs cents.
- **A business site or store expecting real traffic:** same One-Click route, but start at 2 GB of RAM. Resizing later is possible, though the server goes offline during the resize, so starting comfortable is cheaper than it sounds.
- **You never want to see a terminal:** be honest with yourself here. Vultr is unmanaged — SSL, updates, and backups are your job — so either budget the time or look at managed WordPress hosts ($25–50/month for comparable hardware) or a control-panel service layered on top of your Vultr account.

Whichever way you go, grab the security-user credentials from the server information page before you start, and set your domain URLs in Settings → General before you publish a word. [👉 Sign up at Vultr and deploy your first WordPress server](https://bit.ly/VulTR) — the whole thing runs at pennies a day and is reversible whenever you change your mind.
