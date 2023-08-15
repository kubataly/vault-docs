# What is Vault SSH Authentication
The Vault SSH secrets engine provides secure authentication and authorization for access to machines via the SSH protocol. The Vault SSH secrets engine helps manage access to machine infrastructure, providing several ways to issue SSH credentials.

# Risks with SSH Key Based Authentication
 Let’s begin by reviewing the limitations of SSH key-based authentication and what problems we are trying to solve:

- __Risk of private key compromise:__ Users will sometimes mishandle private keys, intentionally or unintentionally exposing them to other users or placing them in insecure locations.
- __Key rotation:__ Revoking private keys is a complex operation. How do you ensure that all copies of the private key are accounted for? What happens when administrators who have made copies of private keys leave the company?
- __Risk of unauthorized access:__ Over time, large collections of keys are created and implemented in multiple systems. Not having an inventory that tracks the usage of these keys, their relationships, what systems they access, and their usage patterns raises the risks of unauthorized access.
- __Scalability and complexity:__ Managing the keys across multiple systems and cloud environments consistently is a complex operation. Different architectures are introduced depending on which cloud environment the hosts are deployed to. The likelihood of incidents due to SSH key mismanagement is growing, and so is the level of harm these incidents can cause. When organizations Google “SSH key management tools” looking for answers, the results are often expensive and complex solutions that weren’t built for the low-trust perimeters of the public cloud. Any service outage in one of these SSH key management tools typically results in no access to any host.

# The Workflow Outline
### Our architecture to implement SSH certificate authentication with HashiCorp Vault looks like this:
![image](https://www.hashicorp.com/_next/image?url=https%3A%2F%2Fwww.datocms-assets.com%2F2885%2F1625670267-ssh-architecture.png&w=3840&q=75)

__The numbers in the diagram represent the following steps:__

1. User creates a personal SSH key pair.
2. User authenticates to Vault with their __OneLogin__ credintials.
3. Once authenticated, the user sends their SSH public key to Vault for signing.
4. Vault signs the SSH key and return the SSH certificate to the user.
5. User initiates SSH connection using the SSH certificate.
6. Host verifies the client SSH certificate is signed by the trusted SSH CA and allows connection.