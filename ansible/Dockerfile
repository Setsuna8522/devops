# Use an official Python runtime as a parent image
FROM python:3.9

# Set environment variables
ENV ANSIBLE_HOST_KEY_CHECKING=False

# Install Ansible and sshpass
RUN pip install ansible
RUN apt-get update && apt-get install -y sshpass

# Set the working directory
WORKDIR /ansible

# Copy the current directory contents into the container
COPY . /ansible/

# Run Ansible playbook
CMD ["ansible-playbook", "deploy.yml"]
