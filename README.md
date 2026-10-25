# Bread Distribution V3 – PPKI SMK Tansau

A simple web app for the PPKI department to run a school bread sale. Each student sells only the buns they made, and the teacher gets a daily money report that carries forward from day to day.

It runs as a single static page on **GitHub Pages**, with **Firebase** (Authentication and Firestore) for logins and data.

## What it does

- **3 teachers**, each with their own classes. A teacher can only see their own classes.
- **Students need no account.** They open a class link and tap their name.
- **Per-student stock.** The teacher sets how many buns each student made. A student can only sell what they made, and the app tracks made, sold and left for each student.
- **Bread type and price per day.** The teacher sets these for each date. They are the same for every student in that class. A day can have more than one bread type.
- **Cash and QR** payments are recorded on every sale, with the buyer's name.
- **Daily report** that carries the balance forward automatically.
- **Lock day** to freeze a finished day.

## Files

| File | Purpose |
|---|---|
| `index.html` | The whole app (teacher and student screens) |
| `firestore.rules` | Security rules to paste into Firebase |
| `README.md` | This file |

## Setup

### 1. Create the Firebase project

1. Go to the [Firebase console](https://console.firebase.google.com) and create a project.
2. Add a **Web app** (the `</>` icon) and copy the config values.
3. Open **Build → Firestore Database** and create a database.
4. Open **Build → Authentication → Sign-in method** and enable **Email/Password** (for teachers). Students do not need any sign-in.
5. Open **Authentication → Settings → Authorized domains** and add your GitHub Pages domain, for example `yourname.github.io`.

### 2. Add the config

In `index.html`, find this line near the top of the script and fill it in:

```js
const CFG = { apiKey:"", authDomain:"", projectId:"", appId:"" };
```

The Firebase web config is not a secret. The security rules below are what protect your data.

### 3. Add the security rules

In **Firestore Database → Rules**, replace the contents with `firestore.rules` and click **Publish**.

### 4. Create the teacher accounts

The app has no sign-up screen, so only accounts you create can sign in. In **Authentication → Users**, click **Add user** and create the 3 teacher accounts with an email and password each.

### 5. Deploy to GitHub Pages

1. Create a GitHub repository and upload `index.html` (and optionally the other files).
2. Go to **Settings → Pages**, choose the `main` branch and the root folder, and save.
3. Open the Pages URL after a minute or two.

## How to use it

### Teacher

1. Sign in with your email and password.
2. Click **Add class**, then **Add student** for each student.
3. Each day, on the **Add bread** tab:
   - Pick the date at the top right.
   - Add the bread types and prices for the day.
   - Enter how many each student made.
   - Click **Save bread and quantities**.
4. Use the **student link** at the top of the screen. Press **Open student page** to go straight to the selling side, or **Copy** to send it to student devices.
5. During the day, check the **Sales today** tab for each student's made, sold and left, and the cash and QR totals. A wrongly recorded sale can be removed there, and the buns go back to that student's stock.
6. At the end of the day, open the **Report** tab, type the **spending**, **salary paid** and any **remarks**, and click **Save report**. Click **Lock day** when you are done.

### Payment QR code

1. In the teacher view, open **Payment QR code** under the student link.
2. Upload a picture of the class's QR code (a screenshot or photo works). The app shrinks it automatically.
3. When a student chooses **QR**, the code appears on their screen so the buyer can scan it. Tapping the code enlarges it.
4. Each class has its own QR code. Use **Replace** or **Remove** to change it.

If you change the QR code, student devices that are already open need a page refresh to show the new one.

### Student

1. Open the class link.
2. Tap your name.
3. Pick the bread, type the buyer's name, set how many, choose **Cash** or **QR**, and press **Record sale**.
4. Open **My sales** to see your own cash, QR and total.
5. If someone else is using the device, tap **Not you?** to switch.

## How the report works

```
Total money = Balance brought forward + Total sales − Salary paid − Spending
```

| Line | Where it comes from |
|---|---|
| Balance brought forward | The previous date's total money (0 for the first day) |
| Total sales | Buns sold × that day's price, summed over the whole class |
| Spending (raw cost) | Typed by the teacher |
| Salary paid | Typed by the teacher |
| Total money | Calculated, and carried to the next day |

## Reports

- **Daily report** (Report tab): title, date, balance brought forward, total money received, total money now, spending, salary paid, current balance, attendance (present and absent names), a per-student table of made, sold and money, cash and QR received, and the teacher's remarks.
- **Monthly report**: pick a month and press **Show report** for a day-by-day table with totals, remarks, and a per-student summary.
- **Yearly report**: pick a year for a month-by-month table with totals and a per-student summary.
- **Export to Excel**: every report has an **Export** button that downloads a formatted `.xlsx` file (Calibri, bordered tables, blue headers).
  - **Daily:** a *Daily report* sheet and a *Sales list* sheet.
  - **Monthly:** the month summary sheet, one sheet for each day that has records, and one *Sales* sheet that lists each day's sales by student.
  - **Yearly:** one summary sheet with a month-by-month table and a by-student table.
  - The export needs an internet connection to load the Excel library.
- Attendance is worked out automatically: a student is present if they were given buns to sell that day.

## Data structure (Firestore)

```
classes/{classId}                 name, teacherUid
  students/{studentId}            name
  days/{date}                     date, breads[], spending, salary, locked,
                                  balance, sales, closing
    stock/{studentId_breadId}     studentId, breadId, made, sold
    sales/{saleId}                studentId, studentName, bread, buyer,
                                  qty, pay (cash/qr), amount, ts
```

## Security notes

- Teachers can read and write only the classes where `teacherUid` matches their own login.
- Student devices need no sign-in or account. They can read the class, add sales, and raise `sold` on a stock record, never above `made`. They cannot change prices, quantities, costs or other classes.
- Students cannot write to a locked day.
- The **student link contains the class ID, which acts as the password**. Anyone with the link can record sales for that class, so share it only with the class. If it leaks, create a new class and move over.

## Known limits

- Students cannot undo their own sale. The teacher removes it from **Sales today**.
- If you change an earlier day after later days exist, the later days' balances only update when the teacher reopens them. Locking each day avoids this.
- Firestore may ask you to create an index the first time a day is opened. Click the link in the browser console error and it sets itself up.
- Teacher sign-in and the student link work on the same site. A teacher who opens a student link on their own device will use the student screen on that device.

## Troubleshooting

| Problem | Likely cause |
|---|---|
| Blank page or console error about config | The `CFG` values in `index.html` are empty or wrong |
| Teacher cannot sign in | The account was not created in Authentication, or Email/Password is not enabled |
| Student link shows an error or "Missing or insufficient permissions" | The updated `firestore.rules` were not published, or the link was copied incorrectly |
| "Missing or insufficient permissions" | The rules were not published, or the teacher is signed in with a different account |
| "The query requires an index" | Click the link in the console error to create it |