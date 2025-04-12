package com.automation.Pages;

import java.util.List;

import org.junit.Assert;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.interactions.Actions;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;

import com.automation.Utilis.DriverUtilis;

import io.cucumber.datatable.DataTable;

public class HomePage extends BasePage {
	@FindBy(id = "branding")

	private WebElement successErrorMsg;

	@FindBy(xpath = "//a[@class='help-icon-div']")
	private WebElement questionField;

	@FindBy(xpath = "//img[@alt='Mobile app QR code']")
	private WebElement imgQRcode;

	@FindBy(xpath = "//table[@id='table1']/tbody/tr")
	private List<WebElement> rowSize;
	@FindBy(xpath = "//b[text()='Admin']")
	private WebElement btnAdmin;

	@FindBy(xpath = "//a[@id='menu_admin_UserManagement']")
	private WebElement btnUserManagement;

	@FindBy(xpath = "//a[text()='Users']")
	private WebElement btnUsers;
	@FindBy(xpath = "//input[@id='btnAdd']")
	private WebElement buttonAdd;

	final String XPATH_DataTable = "//table[@id='table1']/tbody/tr[%s]/td[not(./a)]";

	public void verifyLoginSuccessful() throws InterruptedException {
		/*
		 * WebDriverWait wait=new WebDriverWait(DriverUtilis.getDriver(), 10);
		 * wait.until(ExpectedConditions.titleIs(successErrorMsg));
		 */
		Assert.assertTrue("Login Not Successful", successErrorMsg.isDisplayed());
	}

	public void clickQuestionMark() {
		questionField.click();
		Assert.assertTrue("QR Code Not Present", imgQRcode.isDisplayed());
	}

	public void verifyDataTable(DataTable dataTable) throws InterruptedException {
		List<List<String>> expData = dataTable.asLists();
		for (int i = 0; i < expData.size(); i++) {
			String finalloc = String.format(XPATH_DataTable, i + 1);
			List<WebElement> dataList = driver.findElements(By.xpath(finalloc));
			for (int j = 0; j < expData.get(i).size(); j++) {
				String expectedData = expData.get(i).get(j);
				String actualData = dataList.get(j).getText();
				Thread.sleep(3000);
				Assert.assertEquals(expectedData, actualData);
			}
		}

	}

	public void clickUserManagement() {

		Actions act = new Actions(driver);
		act.moveToElement(btnAdmin).moveToElement(btnUserManagement).build().perform();
		btnUsers.click();
		buttonAdd.click();

	}

	public void clickAddButton() {
		buttonAdd.click();
	}

}
