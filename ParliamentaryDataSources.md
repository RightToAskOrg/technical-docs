# Data sources

## Public authorities
From [RightToKnow.org.au](https://www.righttoknow.org.au/body/list/all)

- [All Australian public Authorities](https://www.righttoknow.org.au/body/all-authorities.csv)

## Parliament of Australia
From [https://www.aph.gov.au/Senators_and_Members/Guidelines_for_Contacting_Senators_and_Members/Address_labels_and_CSV_files](https://www.aph.gov.au/Senators_and_Members/Guidelines_for_Contacting_Senators_and_Members/Address_labels_and_CSV_files):

- List of all [Members of the House of Representatives](https://www.aph.gov.au/-/media/03_Senators_and_Members/Address_Labels_and_CSV_files/FamilynameRepsCSV.csv) plus [pdf with email addresses](https://www.aph.gov.au/-/media/03_Senators_and_Members/32_Members/Lists/Members_List_2021.pdf).
- List of all [Senators](https://www.aph.gov.au/-/media/03_Senators_and_Members/Address_Labels_and_CSV_files/Senators/allsenel.csv) plus [pdf with email addresses](https://www.aph.gov.au/-/media/03_Senators_and_Members/31_Senators/contacts/los.pdf).

See also various schedules, such as pdfs from [Senate Estimates](https://www.aph.gov.au/Parliamentary_Business/Senate_estimates/Next_hearings).

## Assembly of the ACT

- List of all [members of the Assembly](https://www.parliament.act.gov.au/__data/assets/excel_doc/0007/874231/Members-mail-merge-list-2021-0329.xlsx) misses the speaker's electorate.
- Better to parse [HTML](https://www.parliament.act.gov.au/members/members-of-the-assembly)

## Parliament of New South Wales

From [https://www.parliament.nsw.gov.au/members/downloadables/Pages/downloadable-lists.aspx](https://www.parliament.nsw.gov.au/members/downloadables/Pages/downloadable-lists.aspx):

- List of [all members](https://www.parliament.nsw.gov.au/_layouts/15/NSWParliament/memberlistservice.aspx?members=Both&format=Excel)
- List of [Legislative Assembly members](https://www.parliament.nsw.gov.au/_layouts/15/NSWParliament/memberlistservice.aspx?members=LA&format=Excel)
- List of [Legislative Council members](https://www.parliament.nsw.gov.au/_layouts/15/NSWParliament/memberlistservice.aspx?members=LC&format=Excel)

## Legislative Assembly of the Northern Territory

Seems to be only available by selecting [individual members](https://parliament.nt.gov.au/members/by-name) or in pdf:

- List of all [Members of the Legislative Assembly](https://parliament.nt.gov.au/__data/assets/pdf_file/0004/932971/MASTER-List-of-Members-Fourteenth-Assembly-as-at-September-2021.pdf)

## Queensland Parliament

From [https://www.parliament.qld.gov.au/Members/Current-Members/Full-Mailing-Lists](https://www.parliament.qld.gov.au/Members/Current-Members/Full-Mailing-Lists)

- List of all [members of the Assembly](https://documents.parliament.qld.gov.au/Members/mailingLists/MEMMERGEEXCEL.xls)

## Parliament of South Australia

In html:

- List of [All members of the House of Assembly](https://www.parliament.sa.gov.au/en/House-of-Assembly/Members) contains an iframe whose content is dynamically generated from [JSON](https://contact-details-api.parliament.sa.gov.au/api/HAMembersDetails)
- List of [All members of the Legislative Council](https://www.parliament.sa.gov.au/en/Legislative-Council/Members) contains an iframe whose content is dynamically generated from [JSON](https://contact-details-api.parliament.sa.gov.au/api/LCMembersDetails)


## Parliament of Tasmania

From the [House of Assembly](https://www.parliament.tas.gov.au/HA/MainHA.html) and [Legislative Council](https://www.parliament.tas.gov.au/LC/MainLC.html) websites:

- List of all [House of Assembly Members](https://www.parliament.tas.gov.au/Members/HAMembers.xlsx)
- List of all [Legislative Council Members](https://www.parliament.tas.gov.au/members/lcMembers.xlsx)
 
## Parliament of Victoria
From [https://www.parliament.vic.gov.au/about/people-in-parliament/members](https://www.parliament.vic.gov.au/about/people-in-parliament/members):

- List of [all members](https://www.parliament.vic.gov.au/images/members/members.csv)
- List of [Legislative Assembly members](https://www.parliament.vic.gov.au/images/members/assemblymembers.csv)
- List of [Legislative Council members](https://www.parliament.vic.gov.au/images/members/councilmembers.csv)

## Parliament of Western Austraila

Seems to be only html and pdf.

- List of [Members of the Legislative Assembly](https://www.parliament.wa.gov.au/parliament/memblist.nsf/WebCurrentMembLA?OpenView)
- List of [Members of the Legislative Council](https://www.parliament.wa.gov.au/parliament/memblist.nsf/WebCurrentMembLC?OpenView)
