<h1>Apache SSL Configuration</h1>

<h1>Apache SSL Configuration</h1>

<p>The first step is to make some modifications to your <code>httpd.conf</code>:</p>
<pre><code class="language-bash">code /opt/homebrew/etc/httpd/httpd.conf</code></pre>
<p>In this file you should uncomment both the <code>socache_shmcb_module</code>, <code>ssl_module</code>, and also the include for the <code>httpd-ssl.conf</code> by removing the leading <code>#</code> symbol on those lines:</p>
<pre><code class="language-apache">LoadModule socache_shmcb_module lib/httpd/modules/mod_socache_shmcb.so
```
LoadModule ssl_module lib/httpd/modules/mod_ssl.so
```
Include /opt/homebrew/etc/httpd/extra/httpd-ssl.conf</code></pre>
<p>Next we need to change the default <code>8443</code> port to the more standard <code>443</code> and comment out some sample code.  So we need to open the SSL config file:</p>
<pre><code class="language-bash">code /opt/homebrew/etc/httpd/extra/httpd-ssl.conf</code></pre>
<p>find:</p>
<pre><code class="language-apache">Listen 8443</code></pre>
<p>replace it with:</p>
<pre><code class="language-apache">Listen 443</code></pre>
<p>then find:</p>
<pre><code class="language-apache"><VirtualHost _default_:8443>

#   General setup for the virtual host
DocumentRoot "/opt/homebrew/var/www"
ServerName www.example.com:8443</code></pre>
<p>and replace the <code>8443</code> references with <code>443</code> and update the <code>DocumentRoot</code> and <code>ServerName</code> based on the values you setup in your <code>httpd.conf</code> file:</p>
<pre><code class="language-apache"><VirtualHost _default_:443>

#   General setup for the virtual host
DocumentRoot "/Users/your_user/Sites"
ServerName dev.grav.rocks:443</code></pre>
<p>Next locate the <code>Server Certificate:</code> section add the location of your <code>SSLCertificateFile</code> you obtained above:</p>
<pre><code class="language-apache">SSLCertificateFile "/opt/homebrew/etc/certbot/certs/live/dev.grav.rocks/fullchain.pem"</code></pre>
<p>Lastly, locate the <code>Server Private Key:</code> section add the location of your <code>SSLCertificateKeyFile</code> you obtained above:</p>
<pre><code class="language-apache">SSLCertificateKeyFile "/opt/homebrew/etc/certbot/certs/live/dev.grav.rocks/privkey.pem"</code></pre>
<p>Save the file at this point. Then all you need to do now is double check your Apache configuration syntax:</p>
<pre><code class="language-bash">apachectl configtest</code></pre>
<p>If all goes well, restart Apache:</p>
<pre><code class="language-bash">brew services stop httpd
brew services start httpd</code></pre>
<div class="notices tip">
<p>You can <code>tail -f /opt/homebrew/var/log/httpd/error_log</code>, the Apache error log while you restart to see if you have any errors.</p>
</div>
<p>Now simply point your browser at <code>https://dev.grav.rocks</code> and you should see the page loading and a comforting message in the browser address bar about how secure your site is.</p>
<h2>Maintenance &amp; Renewal</h2>
<p>By their nature LetsEncrypt certificates are short-lived that are <a href="https://letsencrypt.org/2015/11/09/why-90-days.html">valid for 90 days</a> only. This means that you need to renew them.  The simplest way to accomplish this is to simply run:</p>
<pre><code class="language-bash">certbot renew
brew services restart httpd</code></pre>
<p>However, you can only run this when you are close to renewal (within 30 days). The best solution is to automate this process by using a cron-job to run the process weekly. First we need to add an entry to the crontab that will run the renewal script every sunday at 3am::</p>
<pre><code class="language-bash">(crontab -l 2>/dev/null; echo "0 3 * * 0 certbot renew --post-hook 'brew services restart httpd' > /dev/null 2>&amp;1") | crontab -</code></pre>
<p>You can check this looks correct by running:</p>
<pre><code class="language-bash">crontab -l</code></pre>
<p>You should see:</p>
<pre><code class="language-bash">0 3 * * 0 certbot renew --post-hook 'brew services restart httpd' > /dev/null 2>&amp;1</code></pre>
<h1>Self-Signed SSL</h1>
<p>Sometimes there are scenarios where you want to mimic an existing site during development, or you don't have a spare domain name but just need to test under SSL.  Either way there are still occasions where a valid SSL certificate is not required, and you just need to get a <strong>self-signed SSL</strong> certificate in place.</p>
<p>We'll install <code>mkcert</code> to serve as our certificate authority (CA), and also <code>nss</code> to ensure firefox can use certificate authority sever.</p>
<pre><code class="language-bash">brew install mkcert nss</code></pre>
<p>Next we have to install the server and run it (enter your password when prompted):</p>
<pre><code class="language-bash">mkcert -install</code></pre>
<p>Let's create a good location for the certificates:</p>
<pre><code class="language-bash">cd /opt/homebrew/etc/httpd
mkdir certs &amp;&amp; cd certs</code></pre>
<p>Now all we have to do is generate a certificate for any domain we wish to use.  For example, you could create one for <code>localhost</code> with:</p>
<pre><code class="language-bash">mkcert localhost</code></pre>
<p>or one for <code>grav-admin.test</code> with: </p>
<pre><code class="language-bash">mkcert grav-admin.test</code></pre>
<p>These commands will create <code>.pem</code> and <code>-key.pem</code> files for each domain.</p>
<h2>Setting up Apache for Self-Signed SSL certificate</h2>
<p>Follow exactly the same steps as the <strong>Apache SSL Configuration</strong> section above to enable SSL in Apache.</p>
<p>When you get to the part where you set the <code>SSLCertificateFile</code> and <code>SSLCertificateKeyFile</code> use the files we just generated rather than the LetsEncrypt <code>.pem</code> files:</p>
<pre><code class="language-apacheconfig">SSLCertificateFile "/opt/homebrew/etc/httpd/certs/grav-admin.test.pem"
SSLCertificateKeyFile "/opt/homebrew/etc/httpd/certs/grav-admin.test-key.pem"</code></pre>
<p>Check the syntax:</p>
<pre><code class="language-bash">/opt/homebrew/bin/httpd -t</code></pre>
<p>Restart the server to have the configuration changes take effect:</p>
<pre><code class="language-bash">brew services restart httpd</code></pre>
<h1>SSL in Apache Virtual Hosts</h1>
<p>The configuration we've outlined in this article focuses on setting a root-level SSL certificate.  You can of course generate multiple unique certificates for various domains and sites.  When you do this, you need to enable SSL for a virtual host via editing the <code>httpd-vhosts.conf</code> file.</p>
<p>If you already have a vhost entry for <code>http</code> on port <code>80</code> for a particular site, you can just copy this and replace <code>80</code> with <code>443</code> then add the entries for your key and certificate.  </p>
<p>Let's assume we have created a <strong>self-signed certificate</strong> for <code>grav-admin.test</code>, and we already have a working vhost configuration for <code>http</code>/port <code>80</code> that we setup in <a href="/blog/macos-sequoia-apache-mysql-vhost-apc">Part 2</a> of the guide:</p>
<pre><code class="language-apacheconfig"><VirtualHost *:80>
    DocumentRoot "/Users/your_user/Sites/grav-admin"
    ServerName grav-admin.test
</VirtualHost></code></pre>
<p>Then we simply copy and paste this and change the port to <code>443</code> and add references to turn on SSL and use the generated SSL files:</p>
<pre><code class="language-apacheconfig"><VirtualHost *:80>
    DocumentRoot "/Users/your_user/Sites/grav-admin"
    ServerName grav-admin.test
</VirtualHost>

<VirtualHost *:443>
    DocumentRoot "/Users/your_user/Sites/grav-admin"
    ServerName grav-admin.test
    SSLEngine on
    SSLCertificateFile "/opt/homebrew/etc/httpd/certs/grav-admin.test.pem"
    SSLCertificateKeyFile "/opt/homebrew/etc/httpd/certs/grav-admin.test-key.pem"
</VirtualHost></code></pre>
<p>Check the syntax:</p>
<pre><code class="language-bash">/opt/homebrew/bin/httpd -t</code></pre>
<p>Restart the server to have the configuration changes take effect:</p>
<pre><code class="language-bash">brew services restart httpd</code></pre>
