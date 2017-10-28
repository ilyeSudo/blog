---
author: Ilyes
layout: post
title:  "Webservices, Hands-On!"
date:   2017-10-25
comments: true
category:
- Web
- Webservice
- SOAP
- Software-Architecture
---

Webservice is a software system designed to support interoperable machine-to-machine interaction over a network. [" wiki "](https://en.wikipedia.org/wiki/Web_service#cite_note-W3WSG-1 "Wikipedia")

## Prerequisite
Before we start, i will claim that you already know what a [Webservice](https://www.tutorialspoint.com/webservices/what_are_web_services.htm) architecture is, when and why they are used for, and you're familiar with [XML](https://www.w3schools.com/xml/xml_services.asp), [WSDL](https://www.w3.org/TR/wsdl), and [SOAP](https://www.tutorialspoint.com/soap/what_is_soap.htm).

If it's not the case, i would rather ask you to have a look on them first. But if so, then let's get started 💪🚀

## Description
The scenario illustrated in the figure below is a loan application.

[<img src="/assets/img/blog/2017-10-25-Webservices-Hands-On/figure1.png" style="height: 80%;width: 80%;"/>](/assets/img/blog/2017-10-25-Webservices-Hands-On/figure1.png "Figure1. Real Estate Loan Application Process")

And here's the [WSDL](https://www.w3.org/TR/wsdl)
{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<xsd:schema xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:tns="http://xml.netbeans.org/schema/HomeLoanApplication" targetNamespace="http://xml.netbeans.org/schema/HomeLoanApplication">
   <xsd:element name="LoanInformation">
      <xsd:complexType>
         <xsd:sequence>
            <xsd:element name="BorrowerInformation">
               <xsd:complexType>
                  <xsd:sequence>
                     <xsd:element name="Name" type="xsd:string" />
                     <xsd:element name="SSN" type="xsd:string" />
                     <xsd:element name="Address" type="tns:AddressType" />
                  </xsd:sequence>
               </xsd:complexType>
            </xsd:element>
            <xsd:element name="LoanDetails">
               <xsd:complexType>
                  <xsd:sequence>
                     <xsd:element name="SSN" type="xsd:string" />
                     <xsd:element name="LoanAmount" type="xsd:string" />
                     <xsd:element name="HomePrice" type="xsd:string" />
                     <xsd:element name="HomeAddress" type="tns:AddressType" />
                  </xsd:sequence>
               </xsd:complexType>
            </xsd:element>
         </xsd:sequence>
      </xsd:complexType>
   </xsd:element>
   <xsd:element name="AppraisalStatus">
      <xsd:complexType>
         <xsd:sequence>
            <xsd:element name="Status" type="xsd:string" />
            <xsd:element name="SSN" type="xsd:string" />
         </xsd:sequence>
      </xsd:complexType>
   </xsd:element>
   <xsd:element name="CreditCheckStatus">
      <xsd:complexType>
         <xsd:sequence>
            <xsd:element name="Status" type="xsd:string" />
            <xsd:element name="SSN" type="xsd:string" />
         </xsd:sequence>
      </xsd:complexType>
   </xsd:element>
   <xsd:complexType name="AddressType">
      <xsd:sequence>
         <xsd:element name="Addr1" type="xsd:string" />
         <xsd:element name="CityState" type="xsd:string" />
         <xsd:element name="Zip" type="xsd:string" />
         <xsd:element name="Telephone" type="xsd:string" />
      </xsd:sequence>
   </xsd:complexType>
</xsd:schema>
{% endhighlight %}

It begins when a user requests a loan by sending a request. The latter then passes through two steps of verification:
* credit check of the applicant (Figure 1: Credit Check Service)
* Amount check: whether the applicant's apartment is worth the amount of the loan requested by the applicant (Figure 1: Home Appraisal Service)
On the basis of the result of these two responses, the loan is approved or refused. The result is communicated to the loan applicant in a synchronous manner.

This example consists of three parts:
- Part 1. Creation of a Credit Check Service.
- Part 2. Creation of an Appraisal Service (Home Appraisal Service).
- Part 3. Establishment of a Loan Approval Service (LAS) that will consist of the two services mentioned in parts 1 and 2.

## Part 1. Credit Check Service
In this part, we'll create a credit check service, which checks the solvency of the applicant.

The information carried by the entry message of the relevant operation of this service includes the name, Social Security Number (SSN), and address of the applicant.

The condition that must be verified by this service is as follows: an applicant with NSS starting with number 6 will be considered solvent. Otherwise, the person is considered unsolvent.

The information carried by the exit message of the relevant operation of this service includes the applicant's Social Security Number (SSN) and the status of the response which can be either "Approved" or "Denied". This information is described by the element XML AppraisalStatus XML schema appended.

## Part 1. Code <>
For the implementation, we'll work with **Java J2EE** since it well supports working with webservices. Besides, the Java version used in this tutorial is **"1.8.0_144"** with **Eclipse Java EE IDE** for Web Developers **Kepler** Service Release 2.

First, we'll create a brand new project of cours, and then we start with creating the model classes **AddressType**, **BorrowerInformation**, and **CreditCheckStatus** as described in the WSDL schema above and group them in a package we call _models_.
{% highlight java %}
public class AddressType {
	private String addr1;
	private String cityState;
	private String zip;
	private String telephone;
}
{% endhighlight %}
{% highlight java %}
public class BorrowerInformation {
	private String nom;
	private String ssn;
	private AddressType address;
}
{% endhighlight %}
{% highlight java %}
public class CreditCheckStatus {
	private String status;
	private String ssn;
}
{% endhighlight %}
Ps: Don't forget to generate the Constructor and the Getters/Setters with the shortcut in Eclipse IDE: **Source->Generate Getters and Setters** and **Source->Generate Constructor using Fields**.

Next, let's move to create the Controller class **VerifyCredit** which will be in a separated package _controllers_.
{% highlight java %}
public class VerifyCredit {
	public CreditCheckStatus verify(BorrowerInformation bi){
		 String status = bi.getSsn().startsWith("6") ? "Approved" : "Denied";
		 CreditCheckStatus ccs = new CreditCheckStatus();
		 ccs.setSsn(bi.getSsn());
		 ccs.setStatus(status);
		 return ccs;
		}
	}
{% endhighlight %}

## Part 2. Home Appraisal Service
In this part, you will create an apartment appraisal service.
The business information carried by the entry message of the relevant operation of this service includes the applicant's NSS, the amount of the loan applied for, the price of the house, and its address. This information is described by the XML LoanDetails element of the given XML schema in the appendix.

The condition that must be verified by this service is the following: If the amount of the loan requested is smaller than the price of the house, the service will give a positive answer (Approved), otherwise, it will give a negative answer (Denied).

The business information carried by the exit message of the relevant operation of this service includes the applicant's Social Security Number (SSN) and the status of the response. Which can be either "Approved" or "Denied". This information is described by the element XML AppraisalStatus XML schema appended.

## Part 2. Code <>
For the second part, we’ll create the model classes just as we did previously : LoanDetails and AppraisalStatus and group them in a package we name _models_.
{% highlight java %}
public class LoanDetails {
	private String ssn;
	private String loanAmount;
	private String homePrice;
	private AddressType homeAddress;
}
{% endhighlight %}
{% highlight java %}
public class AppraisalStatus {
	private String status;
	private String ssn;
}
{% endhighlight %}
Ps: I wish you didn't forget to generate the Constructor with the Getters & Setters XD.

And now, it' time to have the Controller class **AppartementEvaluation** created.
{% highlight java %}
public class AppartementEvaluation {

	public AppraisalStatus evaluate(LoanDetails ld){
		try{
		String status = Integer.parseInt(ld.getLoanAmount())<Integer.parseInt(ld.getHomePrice()) ? "Approved" : "Denied";
		AppraisalStatus as = new AppraisalStatus();
		as.setSsn(ld.getSsn());
		as.setStatus(status);
		return as;
		}catch(Exception e){
			return null;
		}
	}
}
{% endhighlight %}

## Part 3. Loan Approval Service
Here it comes the fun part, when it comes to connect the last webservices with each other.

This time, i won't describe you the scenario of the service, but instead, i'll show you the business information carried by the entry message in XML format how it should look like.

{% highlight xml %}
<BorrowerInformation>
  <Name>John Smith</Name>
  <SSN>666777888</SSN>
  <Address>
    <Addr1>?string?</Addr1>
    <CityState>?string?</CityState>
    <Zip>?string?</Zip>
    <Telephone>?string?</Telephone>
  </Address>
</BorrowerInformation>
<LoanDetails>
  <SSN>666777888</SSN>
  <LoanAmount>400000</LoanAmount>
  <HomePrice>500000</HomePrice>
  <HomeAddress>
    <Addr1>?string?</Addr1>
    <CityState>?string?</CityState>
    <Zip>?string?</Zip>
    <Telephone>?string?</Telephone>
  </HomeAddress>
</LoanDetails>
{% endhighlight %}

Along with an exit message like:
```
"Mr. firstname lastname, your loan demand has been accepted successfully"
```
## Part 3. Code <>
Since we will have a class that interacts with the past two WSs, then the third webservice will be a **Client Webservice**.

{% highlight java %}
public class LoanApprobation {

	public String approbation(VerifyCreditStub.BorrowerInformation bi, AppartementEvaluationStub.LoanDetails ld) throws RemoteException{

		AppartementEvaluationStub aeStub = new AppartementEvaluationStub();
		VerifyCreditStub vcStub = new VerifyCreditStub();
		Evaluate e = new Evaluate();
		Verify v = new Verify();
		v.setBi(bi);
		e.setLd(ld);
		EvaluateResponse resEvaluate = aeStub.evaluate(e);
		VerifyResponse resVerify = vcStub.verify(v);

		String status1 = resEvaluate.get_return().getStatus().toString();
		String status2 = resVerify.get_return().getStatus().toString();

		if(status1.equals(status2)){
			return "Mr."+ v.getBi().getNom() +", your loan demand has been accepted successfully";
		}else{
			return "Mr."+ v.getBi().getNom() +", your loan demand has been rejected";
		}
	}
}
{% endhighlight %}

## Client Test
Let's test our architecture with a third-part client class which communicates with the third service **(Loan Approval Service)**

We'll create 2 persons who will make a loan demand, the first will meet the requirements and the second won't.

{% highlight java %}
public class Client {

	public static void main(String[] args) throws RemoteException, LoanApprobationRemoteExceptionException {

		VerifyCreditStub_BorrowerInformation vcbi = new VerifyCreditStub_BorrowerInformation();
		AppartementEvaluationStub_LoanDetails aesld = new AppartementEvaluationStub_LoanDetails();

		Approbation app_ilyes = createUser(vcbi, aesld, "Walid", "10000000", "6000", "644661649640", null, null, null, null);
		print_result(app_walid);
		Approbation app_redouane = createUser(vcbi, aesld, "Wiliam", "80", "6000", "622200005770", null, null, null, null);
		print_result(app_wiliam);
	}

	public static Approbation createUser(VerifyCreditStub_BorrowerInformation borrower,
						   AppartementEvaluationStub_LoanDetails loan_details,
						   String firstname_lastname,
						   String home_price,
						   String loan_ammount,
						   String ssn,
						   String Addr1,
						   String cityState,
						   String telephone,
						   String zip
							){

	borrower.setNom(firstname_lastname);
	AddressType address = new AddressType();
	address.setAddr1(Addr1);
	address.setCityState(cityState);
	address.setTelephone(telephone);
	address.setZip(zip);
	borrower.setAddress(address);
	borrower.setSsn(ssn);

	loan_details.setHomePrice(home_price);
	loan_details.setLoanAmount(loan_ammount);
	loan_details.setSsn(ssn);
	loan_details.setHomeAddress(address);

	Approbation app = new Approbation();

	app.setBi(borrower);
	app.setLd(loan_details);

	return app;
	}

	public static void print_result(Approbation app) throws RemoteException, LoanApprobationRemoteExceptionException{
		LoanApprobationStub loanApp = new LoanApprobationStub();
		ApprobationResponse response = loanApp.approbation(app);
		System.out.println(response.get_return());
	}
}
{% endhighlight %}

### Output
```
Mr. Walid, your loan demand has been accepted successfully
Mr. Wiliam, your loan demand has been rejected
```
