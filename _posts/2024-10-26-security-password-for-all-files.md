---
title: "[G] Balancing Security and User Experience in Report Distribution"
date: 2024-10-26
---

In our current workflow, we send individual reports via email to users. Due to security concerns, each file sent must be encrypted with a password, which is then sent in a separate email. While this ensures security, it has led to a significant inconvenience for users who receive multiple reports simultaneously.

## **The Problem**

Users are inundated with multiple emails—one containing the reports and another containing the corresponding passwords. For instance, if a user receives 10 reports, they will get 10 separate emails with the reports and 10 additional emails with the passwords. This not only clutters their inbox but also creates a cumbersome experience, as they have to match each report with its corresponding password.

![email-before](/images/2024-10-26-email-before.png)


## **The Proposed Solution**

To improve the user experience (UX), we decided to group all reports into a single email. However, from a technical standpoint, each file still needs to be encrypted with a password. Our initial solution was to send one email containing all 10 reports and another email containing 10 passwords—one for each file.

### **The Trade-off**

The business team raised concerns about this approach, arguing that it would still result in a poor UX. Users would have to open each report individually using different passwords, which could be particularly painful if the number of reports is large.

After considering both security and UX, we decided on a trade-off. We opted to use a single password for all the files in a given email. This means that users will receive one email with all the reports and another email with a single password that unlocks all the files. While this reduces the level of security slightly, it significantly enhances the user experience by simplifying the process of accessing the reports.

![email-after](/images/2024-10-26-email-after.png)


## **Conclusion**

This experience taught us the importance of balancing security with user experience. While security is paramount, it should not come at the expense of usability. By making a calculated trade-off, we were able to improve the user experience while still maintaining a reasonable level of security. This solution not only streamlined our report distribution process but also made it more user-friendly, ultimately leading to higher user satisfaction.