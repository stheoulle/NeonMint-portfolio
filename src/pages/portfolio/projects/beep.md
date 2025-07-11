---
layout:  /src/layouts/ProjectLayout.astro
title: 'Beep'
pubDate: 2025-07-09
description: 'Discord-like chat application built with React and TailwindCSS as a college project.'
languages: ["react", "tailwind", "adonis", "typescript","kubernetes"]
image:
  url: "images/projects/beep.webp"
  alt: "Thumbnail of Beep app."
--- 

# A Discord Alternative by DevOps Students

The Polytech Montpellier DevOps class of 2023-2026 has a "fil rouge" project named **Beep**. This project aims to create a web application that replicates the functionalities of [the Discord platform](https://discord.com/).

<div className='flex justify-center items-center'>
  <img src="https://upload.wikimedia.org/wikipedia/fr/4/4f/Discord_Logo_sans_texte.svg" alt="Discord logo" width={100} />
</div>

## Overview

Beep is designed to offer a comprehensive communication platform for communities, friends, and professionals. It provides a seamless and integrated user experience with a focus on real-time communication and collaboration.

## Functionalities

Like Discord, Beep allows users to:

- Create and manage servers (both private and public)
- Send and receive messages in real-time (text and file sharing)
- Join and participate in voice channels
- Customize user profiles and server settings

The project leverages modern web technologies to ensure performance, scalability, and a responsive design:

- **Frontend:** React, TypeScript, and TailwindCSS
- **Backend:** AdonisJS with PostgreSQL

## Interface

### User Authentication

Beep offers a user-friendly authentication process, including traditional login/signup methods and QR code authentication for enhanced user convenience.

<!-- <div className='flex justify-center items-center'>
  <img src="public/images/projects/beep/login-page.png" alt="Beep login page" />
</div> -->

### User Dashboard

Once logged in, users are greeted with an intuitive interface:

- **Left Sidebar:** Displays the servers the user is part of.
- **Main Area:** Shows the messages exchanged in the current channel, with a pannel to see all participants and their status (online, offline, etc.).
- **Right Sidebar:** Lists the channels available in the selected server.

<!-- <div className='flex justify-center items-center'>
  <img src="public/images/projects/beep/server-page.png" alt="Beep home page" />
</div> -->

## Team Work

The Beep project is developed using an Agile methodology, with the project fragmented into sprint periods. Each sprint focuses on specific features and improvements, ensuring continuous progress and regular updates.

### Team Structure

At the beginning of the project, the class was divided into several teams, each responsible for different aspects of the application:

<div class="text-white">
  <table class="table-auto w-full border border-white">
    <thead>
      <tr>
        <th class="border border-white px-4 py-2">Message Team</th>
        <th class="border border-white px-4 py-2">File Team</th>
        <th class="border border-white px-4 py-2">Voice Team</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td class="border border-white px-4 py-2">5 students</td>
        <td class="border border-white px-4 py-2">5 students</td>
        <td class="border border-white px-4 py-2">5 students</td>
      </tr>
      <tr>
        <td class="border border-white px-4 py-2">
          Responsible for developing the messaging functionalities, including sending and receiving text messages in real time.
        </td>
        <td class="border border-white px-4 py-2">
          Responsible for developing the file sharing functionalities, allowing users to upload and download files in channels.
        </td>
        <td class="border border-white px-4 py-2">
          Responsible for developing the voice channel functionalities, enabling users to communicate via voice in real time.
        </td>
      </tr>
    </tbody>
  </table>
</div>

After this short period, the teams were reorganized to focus on the integration of all functionalities into a single cohesive application. We created Gitlab issues to track the progress of each feature and ensure that all teams were aligned with the overall project goals. Each issues had a status:
<div class="text-white">
  <table class="table-auto w-full border border-white">
    <thead>
      <tr>
        <th class="border border-white px-4 py-2">Status</th>
        <th class="border border-white px-4 py-2">Description</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td class="border border-white px-4 py-2">To Develop</td>
        <td class="border border-white px-4 py-2">The feature needs to be developed by the team.</td>
      </tr>
      <tr>
        <td class="border border-white px-4 py-2">To Accept</td>
        <td class="border border-white px-4 py-2">The issue has been described and needs to be accepted by the team before development begins.</td>
      </tr>
      <tr>
        <td class="border border-white px-4 py-2">Ready</td>
        <td class="border border-white px-4 py-2">The feature is ready to be developed by the team.</td>
      </tr>
      <tr>
        <td class="border border-white px-4 py-2">In Progress (WIP)</td>
        <td class="border border-white px-4 py-2">The feature is currently being developed by the team.</td>
      </tr>
      <tr>
        <td class="border border-white px-4 py-2">In Review</td>
        <td class="border border-white px-4 py-2">The feature has been implemented and is awaiting review by the team.</td>
      </tr>
      <tr>
        <td class="border border-white px-4 py-2">Done</td>
        <td class="border border-white px-4 py-2">The feature has been successfully implemented and is ready for deployment.</td>
      </tr>
    </tbody>
  </table>
</div>

When a feature is in the 'In Review' status, it requires two approvals from team members before it can be marked as 'Done' and merged. To facilitate this process, a team member created a Discord bot that assigns reviewers. If a reviewer does not accept the task within five minutes, the bot automatically selects another student to conduct the review.

## Architecture

The architecture of Beep is designed to be scalable, maintainable, and robust. Key components include:

- **Containerization:** Docker
- **Orchestration:** Kubernetes
- **CI/CD Pipeline:** GitLab CI
- **Deployments:** ArgoCD
- **Monitoring and Logging:** Grafana, Prometheus, and Loki

This setup ensures that Beep can handle a large number of users and provide a reliable service with minimal downtime.

## Future Enhancements

Our roadmap for Beep includes several features and improvements:

- **Microservices Architecture:** Transitioning to a microservices architecture to enhance scalability and maintainability.
- **AI Integration:** Implementing AI features to improve user experience.
- **Mobile Application:** Developing a mobile version of Beep to reach a wider audience.

## Conclusion

Beep is more than just a project; it's a testament to the skills and dedication of the Polytech Montpellier DevOps class of 2023-2026.

🚀 *Developed by Light.*
