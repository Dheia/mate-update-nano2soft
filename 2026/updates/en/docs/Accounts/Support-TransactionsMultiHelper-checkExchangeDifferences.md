### Explanation of the `checkExchangeDifferences` Function

- **Input**: Accepts a journal entry ID or a `TransactionHeader` object, and an options array.
- **Check the existence of the entry and its details**.
- **Determine the posting date** used for the check (can be passed in the options or taken from `relay_date_at` of the entry).
- **If there is no posting date or it is equal to the entry date**, and no `force_check`, we consider that there are no differences.
- **For each detail with a non‑base currency**, we obtain the official rate at the posting date, and calculate the rate difference and the financial difference.
- **Check whether the difference exceeds the threshold**.
- **Aggregate the differences** into a `$differences` array and calculate the total difference.
- **Based on the action specified in `action`**:
  - `'none'`: Only return information about the differences.
  - `'event'`: Fire the `tss.accounts.exchangeDifference` event and return the information.
  - `'entry'` (default): Create a single aggregate difference entry that combines all differences, using the difference account specified in the options or general settings.
- **Return**: An array with the same format as `createJournalEntry`, plus `exchange_entry` if an entry was created.

### Notes:

- The exchange difference account can be passed via the `exchange_difference_account` option; otherwise, the general setting is used.
- Some rules (`balance_check`, `multiplicity`) are disabled to avoid conflicts.
- The function uses `createJournalEntry` to create the new entry, ensuring consistent processing.

With this, we have added a powerful tool to check and create exchange differences for any existing entry, with full flexibility in control.
