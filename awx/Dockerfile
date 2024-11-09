# Use the base AWX EE image
FROM quay.io/ansible/awx-ee:latest

# Switch to root user to fix permissions and install tini
USER root

# Set proper permissions on /tmp and create a new directory for Ansible's temporary files
RUN chmod 1777 /tmp && mkdir -p /tmp/ansible_local_tmp && chmod 777 /tmp/ansible_local_tmp

# Set environment variables for Ansible to use /tmp for local directories
ENV ANSIBLE_LOCAL_TEMP=/tmp/ansible_local_tmp
ENV HOME=/tmp
ENV ANSIBLE_TMP_DIR=/tmp/ansible_local_tmp
ENV ANSIBLE_HOME=/tmp

# Install tini and other necessary packages (optional)
RUN yum install -y tini

# Copy the requirements file and install the required collections
COPY requirements.yml /tmp/requirements.yml
RUN ansible-galaxy collection install -r /tmp/requirements.yml

# Switch back to non-root user for security reasons
USER 1000

# Set tini as the entry point
ENTRYPOINT ["/usr/bin/tini", "--"]

# Command to run ansible-runner
CMD ["/usr/local/bin/ansible-run

