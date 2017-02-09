# Setting Up Mail

Email is something that many projects need, but during development you likely do 
not want to actually send email, but you'd rather have sent mail captured for examination
and released to a real mail server only in certain situations.

To handle these and other situations we recommend [Mailhog](https://github.com/mailhog/MailHog)

## Using MailHog

MailHog can be added as another service with your projects Docker Compose file.

See the mail service defined in the [Outrigger Example Mail Project](https://github.com/phase2/outrigger-examples/tree/master/mail/docker-compose.yml)
that service can be copied into your projects `docker-compose.yml` file, or kept 
separate and started as needed.

Using the configuration from the example you would configure your application 
to use `mail.outrigger.vm:1025` as your SMTP server and you could view captured 
mail (and choose to release it) via the web interface accessible at [http://mail.outrigger.vm:8025](http://mail.outrigger.vm:8025)
