FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive

# Fix hash sum mismatch because I'm on a Mac
# https://stackoverflow.com/questions/67732260/how-to-fix-hash-sum-mismatch-in-docker-on-mac
RUN echo "Acquire::http::Pipeline-Depth 0;" > /etc/apt/apt.conf.d/99custom && \
    echo "Acquire::http::No-Cache true;" >> /etc/apt/apt.conf.d/99custom && \
    echo "Acquire::BrokenProxy    true;" >> /etc/apt/apt.conf.d/99custom

# I mainly follow this tutorial:
# https://akhilsharmaa.medium.com/ubuntu-gui-inside-docker-vnc-server-setup-f601687ec66d
RUN apt-get update && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

RUN --mount=target=/var/lib/apt/lists,type=cache,sharing=locked \
    --mount=target=/var/cache/apt,type=cache,sharing=locked \
    apt-get install -y ubuntu-desktop \ 
    openssh-server \ 
    tightvncserver \
    gnome-panel \ 
    gnome-settings-daemon \ 
    metacity \ 
    nautilus \ 
    gnome-terminal \ 
    sudo \
    dbus-x11 \
    vim

# Create the SSH directory and configure permissions
RUN mkdir /var/run/sshd

# Add a new user 'dockeruser' and set a password
RUN useradd -m -s /bin/bash dockeruser && \
    echo 'dockeruser:26092005' | chpasswd

# Optional: Add the user to the sudoers to allow administrative actions
RUN echo 'dockeruser ALL=(ALL) NOPASSWD: ALL' > /etc/sudoers.d/dockeruser && \
    chmod 0440 /etc/sudoers.d/dockeruser

# Enable password authentication in the SSH configuration
RUN sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config

# Optional: Disable root login via SSH
RUN echo "PermitRootLogin no" >> /etc/ssh/sshd_config

# My friend helped me with this part
USER dockeruser
RUN mkdir -p /home/dockeruser/.vnc && \
    echo "26092005" | vncpasswd -f > /home/dockeruser/.vnc/passwd && \
    chmod 600 /home/dockeruser/.vnc/passwd

# Setting up to start
RUN echo "# /bin/sh \n \
    export XKL_XMODMAP_DISABLE=1 \n \
    unset SESSION_MANAGER \n \
    unset DBUS_SESSION_BUS_ADDRESS \n \
    [ -x /etc/vnc/xstartup  ] && exec /etc/vnc/xstartup \n \
    [ -r $HOME/ .Xresources ] && xrdb $HOME/ .Xresources \n \
    xsetroot -solid grey \n \
    vncconfig -iconic & \n \ 
    gnome-panel & \n \
    gnome-settings-daemon & \n \
    metacity & \n \
    nautilus & \n \
    gnome-terminal &" \ 
    > /home/dockeruser/.vnc/xstartup && \
    chmod +x /home/dockeruser/.vnc/xstartup

USER root

# Expose the SSH & VNC port
EXPOSE 22
EXPOSE 1
EXPOSE 5901

# Run the SSH server    
CMD ["/usr/sbin/sshd", "-D"]
