---
title: Expertise - Shivam Khattar
display: Expertise
description: My career path and education
plum: true
wrapperClass: 'text-center'
# cspell:disable
projects:
  Languages:
    - name: 'Java'
      link: 'https://www.java.com/it/'
      desc: 'Classical OOP with garbage collection'
      icon: 'i-mdi:language-java'
    - name: 'Kotlin'
      link: 'https://kotlinlang.org/'
      desc: 'Modern JVM development'
      icon: 'i-simple-icons:kotlin'
    - name: 'Golang'
      link: 'https://go.dev/'
      desc: 'General purpose development'
      icon: 'i-fa6-brands:golang'
    - name: 'Python'
      link: 'https://www.python.org/'
      desc: 'Data analysis and fast development'
      icon: "i-mdi:language-python"
    - name: 'Javascript'
      link: 'https://github.com/slidevjs/slidev'
      desc: 'Flexibility from backend to frontend'
      icon: "i-mdi:language-javascript"
    - name: 'Typescript'
      link: 'https://www.typescriptlang.org/'
      desc: 'Ensure type safety in Javascript'
      icon: "i-mdi:language-typescript"
  Frameworks:
    - name: 'Spring Boot'
      link: 'https://spring.io/projects/spring-boot'
      desc: 'Java framework for rapid development'
      icon: "i-simple-icons:springboot"
    - name: 'Quarkus'
      link: 'https://quarkus.io/'
      desc: 'Supersonic Subatomic Java'
      icon: "i-simple-icons:quarkus"
    - name: 'React'
      link: 'https://react.dev/'
      desc: 'Library for building UIs'
      icon: "i-simple-icons:react"
  Data:
    - name: 'MySQL'
      link: 'https://www.mysql.com/'
      desc: 'Classical relational database'
      icon: "i-tabler:brand-mysql"
    - name: 'PostgreSQL'
      link: 'https://www.postgresql.org/'
      desc: 'More than just a relational database'
      icon: "i-akar-icons:postgresql-fill"
    - name: 'MongoDB'
      link: 'https://www.mongodb.com/'
      desc: 'Flexible NoSQL database database'
      icon: "i-simple-icons:mongodb"
    - name: 'DynamoDB'
      link: 'https://aws.amazon.com/dynamodb/'
      desc: 'Serverless NoSQL database'
      icon: "i-simple-icons:amazondynamodb"
    - name: 'Redis'
      link: 'https://redis.io/'
      desc: 'In-memory data store'
      icon: "i-simple-icons:redis"
    - name: 'Kafka'
      link: 'https://kafka.apache.org/'
      desc: 'Event Driven Development with ease'
      icon: "i-simple-icons:apachekafka"
  Cloud:
    - name: 'AWS'
      link: 'https://aws.amazon.com/'
      desc: 'Cloud computing services'
      icon: "i-fa-brands:aws"
    - name: 'Cloudformation'
      link: 'https://aws.amazon.com/cloudformation/'
      desc: 'Speed up cloud provisioning with IAC'
      icon: "i-carbon:infrastructure-classic"
    - name: 'Terraform'
      link: 'https://www.terraform.io/'
      desc: 'Speed up cloud provisioning with IAC'
      icon: "i-simple-icons:terraform"
    - name: 'Docker'
      link: 'https://www.docker.com/'
      desc: 'Deploy in a deterministic way'
      icon: "i-mdi:docker"
    - name: 'Kubernetes'
      link: 'https://kubernetes.io/'
      desc: 'Container orchestration'
      icon: "i-mdi:kubernetes"
    - name: 'Helm'
      link: 'https://helm.sh/'
      desc: 'Package manager for Kubernetes.'
      icon: "i-mdi:helm"
  CI/CD:
    - name: 'Gitlab'
      link: 'https://about.gitlab.com/'
      desc: 'Git and CI/CD platform'
      icon: "i-mdi:gitlab"
    - name: 'Jenkins'
      link: 'https://www.jenkins.io/'
      desc: 'Continuous integration and automation'
      icon: "i-simple-icons:jenkins"
# cspell:enable
---

<!-- @layout-full-width -->

<ListProjects :projects="frontmatter.projects" />
