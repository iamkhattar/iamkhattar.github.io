---
title: Experience - Shivam Khattar
display: Experience
description: My career path and education
plum: true
wrapperClass: 'text-center'
projects:
  Work Experience:
    - role: 'VP Software Engineer'
      link: 'https://www.barclays.co.uk/'
      company: "Barclays"
      timePeriod: 'Mar 25 - Present'
      location: 'Glasgow, UK'
      icon: 'i-simple-icons:barclays'
    - role: 'AVP Java Reference Architecture Engineer'
      link: 'https://www.barclays.co.uk/'
      company: "Barclays"
      timePeriod: 'Nov 22 - Mar 25'
      location: 'Glasgow, UK'
      icon: 'i-simple-icons:barclays'
    - role: 'Software Engineer'
      link: 'https://www.barclays.co.uk/'
      company: "Barclays"
      timePeriod: 'Aug 20 - Oct 22'
      location: 'Glasgow, UK'
      icon: 'i-simple-icons:barclays'
  Education:
    - role: 'BSc (Hons) Computer Science'
      link: 'https://www.strath.ac.uk/'
      company: "University of Strathclyde"
      timePeriod: 'Aug 17 - Jun 20'
      location: 'Glasgow, UK'
      icon: 'i-ion:university'
    - role: 'Foundation Engineering and Sciences'
      link: 'https://www.strath.ac.uk/'
      company: "University of Strathclyde"
      timePeriod: 'Jan 17 - Aug 17'
      location: 'Glasgow, UK'
      icon: 'i-ion:university'
    - role: 'Indian School Certificate'
      link: 'https://theheritageschool.org/'
      company: "The Heritage School"
      timePeriod: 'Apr 14 - Apr 16'
      location: 'Kolkata, India'
      icon: 'i-ion:school'
    - role: 'Indian Certificate of Secondary Education'
      link: 'https://theheritageschool.org/'
      company: "The Heritage School"
      timePeriod: 'Apr 04 - Apr 14'
      location: 'Kolkata, India'
      icon: 'i-ion:school'
  Certificates:
    - role: 'AWS Certified Solution Architect'
      link: 'https://www.credly.com/badges/a76b8d52-1eeb-4cbc-9d35-067378fd74a9'
      icon: 'i-fa-brands:aws'
    - role: 'AWS Certified Developer'
      link: 'https://www.credly.com/badges/c91d76d8-951e-4834-be07-f7f0d6819d41'
      icon: 'i-fa-brands:aws'
    - role: 'AWS Certified Cloud Practitioner'
      link: 'https://www.credly.com/badges/4c8c11ff-0178-4c42-9bfb-cd56417b1a7e'
      icon: 'i-fa-brands:aws'

---

<!-- @layout-full-width -->

<ListExperiences :projects="frontmatter.projects" />
