Installing a self-signed (gasp!) SSL certificate on my VPS was extremely simple, free, and took less than 5 minutes.

---

Once again, DigitalOcean was the top result on my search for adding a self-signed SSL certificate to my VPS to enable a secure connection. Their guide, [How To Create an SSL Certificate on Nginx for Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-create-an-ssl-certificate-on-nginx-for-ubuntu-14-04), is well-written and very, very simple. The resulting connection scores a whopping "T" on SSL Labs.

If this site actually had logins or any sort of user information, the next step would be buying a proper certificate from namecheap. For now, though, self-signed SSL for those who don't mind the untrusted certificate and HTTP for everyone else will do.