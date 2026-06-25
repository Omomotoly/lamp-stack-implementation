# LAMP Stack Implementation: Step-by-Step Web Infrastructure Deployment

A complete, hands-on implementation of a LAMP (Linux, Apache, MySQL, PHP) stack configured from scratch on an Ubuntu server. This repository documents the foundational setup required to host dynamic web applications.

## 📋 Architectural Overview
The LAMP stack is a classic foundational framework for web development, consisting of four essential layers:
*   **Operating System:** Linux (Ubuntu) 
*   **Web Server:** Apache HTTP Server 
*   **Database:** MySQL 
*   **Scripting Language:** PHP 

## 🚀 Key Implementation Steps

### 1. Environment Provisioning (Linux)
*   Deployed and configured an Ubuntu Server instance.
*   Updated system package repositories and adjusted network security groups to allow HTTP (Port 80) traffic.

### 2. Web Server Configuration (Apache)
*   Installed Apache and verified its active service status.
*   Configured firewall rules to manage inbound traffic.
*   Validated deployment via the default Apache landing page.

### 3. Database Layer Setup (MySQL)
*   Installed MySQL Server for data persistence.
*   Executed `mysql_secure_installation` to enforce strong password policies, disable remote root login, and remove anonymous users.

### 4. Backend Integration (PHP)
*   Installed PHP and its necessary extensions to process server-side scripts.
*   Modified Apache’s directory index configuration (`dir.conf`) to give PHP files execution priority.
*   Validated the entire stack's functionality using a dynamic PHP info script.

## 💡 Key Takeaway
This project reinforces essential infrastructure skills including port configurations, system permissions, and backend service dependencies. It establishes a solid baseline before moving into modern architectural variants.