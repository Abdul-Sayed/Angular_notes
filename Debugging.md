# Angular Debugging

In dev tools, open sources tab and enter CTRL P, and search for the component
Add breakpoints where code likely breaks, and refresh

At each breakpoint, check to confirm what variables are accessable up to that point, hit next to proceed through the breakpoints until the error is found.

To quickly check the data, render it in the view with {{ dataObj | json }}

Try using pipable rxjs operators instead of subscribe in case your forget to unsubscribe

Always check the network tab for each http call to ensure the json contracts for the data sent and response revieved are correct

Run ng lint before pushing code and fix all your linting issues first.
