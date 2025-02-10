import { test, expect } from '@playwright/test';

test('test', async ({ page }) => {

  await page.goto('https://hrisdev-fe.cosmo-testing.com/login');
  await page.locator('input[name="username"]').click();
  await page.locator('input[name="username"]').fill('Admin');
  await page.locator('input[name="password"]').click();
  await page.locator('input[name="password"]').fill('12345');
  await page.getByRole('button', { name: 'Log In' }).click();
  await page.locator('div:nth-child(3) > a').click();
  await page.getByRole('link', { name: 'Job Competency Type' }).click();

  // Add a value
  await page.getByRole('table').getByRole('textbox').click();
  await page.getByRole('table').getByRole('textbox').fill('Job Type 6');
  await page.getByRole('row', { name: 'Job Type' }).getByRole('button').first().click();
  await page.getByText('× Competency Type added').click();
  await page.locator('div').filter({ hasText: '× Competency Type added' }).first().dblclick();

  // Validate the prompt message
  const promptMessage = page.getByText('Record successfully saved');
  expect(promptMessage).not.toBeNull();

  // Verify the added value
  const addedValue = page.getByRole('row', { name: 'Job Type 6' });
  expect(addedValue).not.toBeNull();

  // Edit the added value
  await page.getByRole('row', { name: 'Job Type 6' }).getByRole('button', { name: 'Edit' }).click();
  const editTextbox = page.getByRole('textbox', { name: 'Job Competency Type' });
  await editTextbox.fill('Job Type 6 Edited');
  await page.getByRole('button', { name: 'Save' }).click();

  // Validate the edit prompt message
  const editPromptMessage = page.getByText('Record successfully updated');
  expect(editPromptMessage).not.toBeNull();

  // Verify the edited value
  const editedValue = page.getByRole('row', { name: 'Job Type 6 Edited' });
  expect(editedValue).not.toBeNull();

  // Edit the value to be blank and check for required validation
  await page.getByRole('row', { name: 'Job Type 6 Edited' }).getByRole('button', { name: 'Edit' }).click();
  await editTextbox.fill('');
  await page.getByRole('button', { name: 'Save' }).click();

  // Validate the required field prompt message
  const requiredFieldPromptMessage = page.getByText('Please enter Job Competency Type');
  expect(requiredFieldPromptMessage).not.toBeNull();

  // Verify the textbox has a max length of 250
   const maxLength = await editTextbox.getAttribute('maxlength');
   expect(maxLength).toBe('250');
 
  // Enter a value with more than 250 characters
   const longValue = 'A'.repeat(251);
   await editTextbox.fill(longValue);
   const textboxValue = await editTextbox.inputValue();
   expect(textboxValue.length).toBe(250); // Verify that the value is truncated to 250 characters

  // Add a duplicate value
  await page.getByRole('table').getByRole('textbox').click();
  await page.getByRole('table').getByRole('textbox').fill('Job Type 6');
  await page.getByRole('row', { name: 'Job Type' }).getByRole('button').first().click();

  // Validate the duplicate record message
  const duplicatePromptMessage = page.getByText('Duplicate Job Competency Type is not allowed.');
  await expect(duplicatePromptMessage).toBeVisible();

  // Delete the added value
  await page.getByRole('row', { name: 'Job Type 6 Edited' }).getByRole('button', { name: 'Delete' }).click();
  await page.getByRole('button', { name: 'Yes' }).click();

  // Validate the deletion prompt message
  const deletePromptMessage = page.getByText('Record successfully deleted');
  expect(deletePromptMessage).not.toBeNull();

  // Verify the value is deleted
  const deletedValue = await page.getByRole('row', { name: 'Job Type 6 Edited' }).count();
  expect(deletedValue).toBe(0);
});


