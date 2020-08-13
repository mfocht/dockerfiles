FROM olbat/debian:testing

# Install Packages (basic tools, cups, basic drivers, HP drivers)
RUN apt-get update \
&& apt-get install -y \
  sudo \
  whois \
  cups \
  cups-client \
  cups-bsd \
  cups-filters \
  foomatic-db-compressed-ppds \
  printer-driver-all \
  openprinting-ppds \
  hpijs-ppds \
  hp-ppd \
  hplip \
  smbclient \
  printer-driver-cups-pdf \
  ghostscript \
&& apt-get clean \
&& rm -rf /var/lib/apt/lists/*

# Add user and disable sudo password checking
RUN useradd \
  --groups=sudo,lp,lpadmin \
  --create-home \
  --home-dir=/home/print \
  --shell=/bin/bash \
  --password=$(mkpasswd print) \
  print \
&& sed -i '/%sudo[[:space:]]/ s/ALL[[:space:]]*$/NOPASSWD:ALL/' /etc/sudoers

# Configure the service's to be reachable
RUN /usr/sbin/cupsd \
  && while [ ! -f /var/run/cups/cupsd.pid ]; do sleep 1; done \
  && cupsctl --remote-admin --remote-any --share-printers \
  && kill $(cat /var/run/cups/cupsd.pid)

# Patch the default configuration file to only enable encryption if requested
RUN sed -e '0,/^</s//DefaultEncryption IfRequested\n&/' -i /etc/cups/cupsd.conf

#Allow Remote Administration of CUPS
#
#RUN echo '<Location />
#    Order allow,deny
#    Allow localhost
#    Allow from 192.168.0.*
#    Allow from 10.0.*.*
#</Location>

#Listen 0.0.0.0:631'
#>> /etc/cups/cupsd.conf
#
#Setup Mobility Print
RUN wget -O pc-mobility-print.sh https://www.papercut.com/products/ng/mobility-print/download/server/linux/
RUN chmod +x pc-mobility-print.sh
RUN useradd -m -d /home/papercut papercut
RUN su - papercut -c 'sh /pc-mobility-print.sh --non-interactive -v'
RUN sudo /home/papercut/pc-mobility-print/MUST-RUN-AS-ROOT
# Default shell
CMD ["/usr/sbin/cupsd", "-f"]
