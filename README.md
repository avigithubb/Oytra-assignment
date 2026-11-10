# Oytra-assignment

## Task 2 Explanation and completion:

- Used github API i.e. https://api.github.com/search/repositories to fetch the repositories from github using n8n's fetch action.
- Filtered top 5 records according to the stars.
- Fetched top repository Readme.md file along with its content.
- stored the top 5 repositories in a single object called repos.
- Implemented a check to know if the current repository have stars more than 1000 or not.
- If it does then formed a message including the details of the repository and the latest news.
- Sent the formed message to Gmail.
- If the repository doesn't have enough stars then it falls back to the failure action where we created a message for the failure mail and sent it to the email.

<img width="1500" height="435" alt="Screenshot 2026-11-10 151650" src="https://github.com/user-attachments/assets/7477b510-139b-4604-8ffa-47b36f300b9f" />
<img width="1450" height="566" alt="Screenshot 2026-11-10 151720" src="https://github.com/user-attachments/assets/5853c1e8-97cc-4470-8c06-da4183e96848" />
