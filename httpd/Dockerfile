# Use the official httpd image as a base
FROM httpd:2.4

# Copy the index.html file into the container
COPY ./index.html /usr/local/apache2/htdocs/

# Copy the updated configuration file
COPY ./httpd.conf /usr/local/apache2/conf/httpd.conf

# The port number the container should expose
EXPOSE 80