# User Stories

## Must have
- As a user, I want to be able to authenticate with the 6 essential CRUD operations on my account. (sign up, login, logout, delete account, update username, update password)
- As a user, I want to be able to save job listings to my account so that I can view them later.
- As a user I want to be able to view a list of all my saved job listings, sorted by date saved with the recommended structure of "Company Name (Job Title) - Date Saved".
  - *the first line is always the "Company Name (Job Title)", leaving it to users' discretion to add a heading or not*
- As a user, I want to be able to create, read and delete keyword counts in my job listings such as "python: 5" or "data engineer: 10" so that I can find jobs that match my skills and interests.
  - *the keyword counts a maximum of 1 value per job listing e.g. if a job listing mentions python 4 times, it will only count as 1 keyword*
  - *the keyword counts update automatically when a job listing is saved*
- As a user, I need to subscribe for "£1.99 a month" to use the service via stripe.
- As a user I need to view a landing page explaining the service and how it works.
- As a user I need to view a pricing page explaining the service and how it works.

## Should have
- As a user, I want to be able to view a filtered list of job listings based on my tracked keywords.
- As a user I want a keyboard shortcut "ctrl+v" to quickly paste the job listing.
- As a user, I want to be able to colour code my keywords so I can identify skills I have vs skills I am acquiring.
- As a user I want to be able to delete unwanted keywords as an overriding function if I want to.

## Could have
- As a user I could have an AI feature that automatically adds "company (job title)" to the job listing heading.
