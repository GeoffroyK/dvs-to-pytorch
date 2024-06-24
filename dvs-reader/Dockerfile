#Download base image ubuntu 20.04
FROM ubuntu:20.04

#Labels
LABEL maintainer="geo"
LABEL version="0.1"
LABEL description="Custom Ubuntu with ROS Noetic Installation for reading events in a tensor-like format"

# Disable Prompt During Packages Installation
ARG DEBIAN_FRONTEND=noninteractive

# Update Ubuntu Software repository
RUN apt update

# Install nginx, php-fpm and supervisord from ubuntu repository
RUN apt install -y nginx php-fpm supervisor && \
    rm -rf /var/lib/apt/lists/* && \
    apt clean

#Define the ENV variable
ENV nginx_vhost /etc/nginx/sites-available/default
ENV php_conf /etc/php/7.4/fpm/php.ini
ENV nginx_conf /etc/nginx/nginx.conf
ENV supervisor_conf /etc/supervisor/supervisord.conf

# Enable PHP-fpm on nginx virtualhost configuration
COPY default ${nginx_vhost}
RUN sed -i -e 's/;cgi.fix_pathinfo=1/cgi.fix_pathinfo=0/g' ${php_conf} && \
    echo "\ndaemon off;" >> ${nginx_conf}

#Copy supervisor configuration
COPY supervisord.conf ${supervisor_conf}

RUN mkdir -p /run/php && \
    chown -R www-data:www-data /var/www/html && \
    chown -R www-data:www-data /run/php

# Volume configuration
VOLUME ["/etc/nginx/sites-enabled", "/etc/nginx/certs", "/etc/nginx/conf.d", "/var/log/nginx", "/var/www/html"]

# Copy start.sh script and define default command for the container
COPY start.sh /start.sh
CMD ["./start.sh"]

# Expose Port for the Application 
EXPOSE 80 443


# ROS Noetic
RUN apt-get -qq update && apt-get install -y curl lsb-release python3-pip git wget \
      && curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key  -o /usr/share/keyrings/ros-archive-keyring.gpg \
      && echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" | tee /etc/apt/sources.list.d/ros-latest.list > /dev/null

RUN apt-get -qq update && apt-get -qq dist-upgrade && apt-get install -y ros-noetic-desktop-full\
                       ros-noetic-camera-info-manager\
                       ros-noetic-image-view

RUN echo "source /opt/ros/noetic/setup.bash" >> ~/.bashrc
SHELL ["/bin/bash", "-c"]
RUN source ~/.bashrc

# Install zurich dvs Driver
RUN apt-get update

#iniviation dvs drivers
RUN sudo apt-get install -y software-properties-common
RUN sudo add-apt-repository -y ppa:inivation-ppa/inivation
RUN apt-get update

RUN apt-get install -y python3-catkin-tools python3-osrf-pycommon

RUN cd /home/
RUN mkdir -p catkin_ws/src
RUN cd catkin_ws
RUN catkin config --init --mkdirs --extend /opt/ros/noetic --merge-devel --cmake-args -DCMAKE_BUILD_TYPE=Release

WORKDIR /home/catkin_ws/src/

#RUN cd /home/catkin_ws/src
RUN git clone https://github.com/catkin/catkin_simple.git
RUN git clone https://github.com/uzh-rpg/rpg_dvs_ros.git
#RUN catkin build dvs_ros_driver
#RUN catkin build dvs_renderer
RUN echo "source /home/catkin_ws/devel/setup.bash" >> ~/.bashrc

