# Handling Complex UI

## 1. What It Is

Complex UI means browser behaviors beyond simple click/fill/assert: iframes, dialogs,
new tabs/popups, file upload/download, shadow DOM, drag-drop, keyboard shortcuts, and
dynamic overlays.

## 2. Why It Matters In Interviews

These are practical-round favorites because they prove real Playwright experience. A
candidate who only knows basic clicks gets stuck here.

## 3. Complete Topic Breakdown

| UI case | Playwright surface |
|---|---|
| iframe | `frameLocator()` |
| alert/confirm/prompt | `page.on('dialog')` or `page.once('dialog')` |
| new tab/popup | `page.waitForEvent('popup')` |
| download | `page.waitForEvent('download')` |
| upload | `locator.setInputFiles()` |
| shadow DOM | normal locators pierce open shadow roots |
| drag-drop | `locator.dragTo()` |
| keyboard | `locator.press()`, `page.keyboard` |
| overlay/modal | role locators + actionability/assertions |

## 4. Manual Testing Angle

Manual testers naturally validate these flows because users hit them directly: upload a
file, export report, open a new tab, accept a confirmation, work inside embedded content,
or use keyboard behavior.

## 5. Automation Angle

Use Playwright's specific API for each browser behavior instead of forcing clicks.

Important trade-off: complex UI tests are often higher maintenance, so automate critical
flows and keep edge cases focused.

## 6. Exact Surface To Remember

Iframe:

```typescript
const frame = page.frameLocator('#report-frame');
await frame.getByRole('button', { name: 'Run' }).click();
```

Dialog:

```typescript
page.once('dialog', async dialog => {
  expect(dialog.message()).toContain('Delete');
  await dialog.accept();
});
await page.getByRole('button', { name: 'Delete' }).click();
```

Popup:

```typescript
const popupPromise = page.waitForEvent('popup');
await page.getByRole('link', { name: 'Open report' }).click();
const popup = await popupPromise;
await expect(popup).toHaveURL(/report/);
```

Download:

```typescript
const downloadPromise = page.waitForEvent('download');
await page.getByRole('button', { name: 'Export' }).click();
const download = await downloadPromise;
await download.saveAs('downloads/report.csv');
```

Upload:

```typescript
await page.getByLabel('Upload file').setInputFiles('fixtures/input.csv');
```

Shadow DOM:

```typescript
await page.getByText('Details').click();
```

## 7. Real Project Example

> "In BASES_PLAYWRIGHT, export/download and modal confirmation flows are good examples.
> I start the event wait before the user action, then assert the downloaded file or popup
> behavior. For iframes, I use `frameLocator` instead of switching driver context like in
> Selenium."

## 8. Common Mistakes / Weak Areas

- Starting popup/download wait after clicking.
- Using Selenium-style frame switching language.
- Handling dialog after the action.
- Assuming XPath pierces shadow DOM.
- Testing upload by typing into hidden input manually.
- Using `force` instead of waiting for modal/overlay state.

## 9. 30-Second Answer

For complex UI, I use the Playwright API built for that browser behavior:
`frameLocator` for iframes, `page.once('dialog')` for dialogs, `waitForEvent('popup')`
for tabs, `waitForEvent('download')` for downloads, and `setInputFiles` for upload. Event
waits must start before the action.

## 10. 2-Minute Answer

Complex UI is where Playwright is stronger than simple WebDriver-style scripting. It has
specific APIs for iframes, dialogs, popups, downloads, uploads, shadow DOM, drag-drop,
and keyboard behavior. I avoid generic sleeps or forced clicks and wait for the actual
browser event.

The important pattern is pre-wait: create the event promise first, perform the action,
then await the event. The trade-off is that complex UI tests can be brittle if they cover
too many edge cases, so I automate critical flows and keep selectors/events precise.

## 11. Interview Q&A

### Q: How do you handle iframe?

Use `frameLocator('#frame').getByRole(...).click()`. In Playwright, you do not switch
driver context like Selenium.

### Q: How do you handle alert?

Register `page.once('dialog', handler)` before the action that opens it.

### Q: How do you handle download?

Start `page.waitForEvent('download')`, click export, then await the download.

### Q: Does Playwright support shadow DOM?

Open shadow DOM works with normal locators. XPath does not pierce shadow roots, and closed
shadow roots are not supported.

## 12. Quick Revision Points

- Use the specific API.
- Event wait before action.
- `frameLocator`, not switchTo frame.
- Upload uses `setInputFiles`.
- Open shadow DOM works with locators.
