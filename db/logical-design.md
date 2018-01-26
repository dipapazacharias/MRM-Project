
	|===  Anasynthesi Project  ===|



------ SQL DB logical design ------



/*  Strong Entity Types  */

Structure (
	structureNo,
	structureName,
	structureType,
	dateFounded,
	dateDeprecated,
	directedBy,
	/structureLevel)
	_Primary Key: structureNo
	_Foreign Key: directedBy	references Structure(structureNo) – DirectedBy		ON UPDATE CASCADE	ON DELETE SET NULL


Member (
	memberNo,
	lastName,
	firstName,
	email,
	dateOfBirth,
	gender,
	membershipStatus,
	comments,
	password,
	coordinator,
	Derived Attributes:
		/age  =  CURRENT DATE – dob,
		/numberOfMeetingAbsences  =  SUM(IsAbsentIn WHERE memberNo = memberNo,
		/viewLevel  =  FROM Has WHERE EXISTS structureNo FROM Sructure 	WHERE structureType = A,B,C,D )
	_Primary Key: memberNo
	_Alternate Key: email
	_Foreign Key: coordinator references Structure (structureNo) – Coordinates		ON UPDATE CASCADE	ON DELETE SET NULL


District (districtNo, districtName, municipalityNo)
	_Primary Key: districtNo
	_Foreign Key: municipalityNo references Municipality (municipalityNo) – IsInMunicipality		NOT NULL	ON UPDATE CASCADE	ON DELETE NO ACTION

Municipality (municipalityNo, municipalityName, countyNo)
	_Primary Key: municipalityNo
	_Foreign Key: countyNo references County (countyNo) – IsInCounty		NOT NULL 	ON UPDATE CASCADE	ON DELETE NO ACTION

County (countyNo, countyName, areaNo)
	_PrimaryKey: countyNo
	_Foreign Key: areaNo references Area (areaNo) – IsInArea		NOT NULL 	ON UPDATE CASCADE	ON DELETE NO ACTION

Area (areaNo, areaName, countryNo)
	_Primary Key: areaNo
	_Foreign Key: countryNo references Country (countryNo) – IsInCountry		NOT NULL 	ON UPDATE CASCADE	ON DELETE NO ACTION

Country (countryNo, countryName)
	_PrimaryKey: countryNo

Occupation (occupationNo, occupationName)
	_PrimaryKey: occupationNo

SocialGroup (socialGroupNo, socialGroupName)
	_Primary Key: socialGroupNo

Meeting (meetingNo, structureNo, meetingDate)
	_Primary Key: meetingNo
	_Alternate Key: structureNo, meetingDate
	_Foreign Key: structureNo 	references Structure(structureNo) – Does		NOT NULL	ON UPDATE CASCADE	ON DELETE NO ACTION

Payment (
	paymentNo,
	memberNo,
	paymentDate,
	paymentType,
	amount,
	payHow,
	paymentReason)
	_Primary Key: paymentNo
	_Foreign Key: memberNo	references Member (memberNo) – DoneBy		NOT NULL ON UPDATE CASCADE ON DELETE NO ACTION



/*  Weak EntityTypes  */

Place (
	districtNo,
	placeComments,
	placeType,
	addressRoad,
	addressNumber,
	memberNo)
	_Primary Key: memberNo, placeType
	_Foreign Key: memberNo	references Member(memberNo) – LivesIn		NOT NULL	ON UPDATE CASCADE	ON DELETE CASCADE
	_Foreign Key: districtNo references District (districtNo) – IsInDistrict		NOT NULL	ON UPDATE CASCADE	ON DELETE NO ACTION



/*  Many-to-Many Relationships  */

HasMembers (memberNo, structureNo)
	_Primary Key: memberNo, structureNo
	_Foreign Key: memberNo 	references Member(memberNo)		ON UPDATE CASCADE	ON DELETE CASCADE
	Foreign Key: structureNo 	references Structure(structureNo)		NOT NULL	ON UPDATE CASCADE	ON DELETE NO ACTION


HasOccupation (memberNo, occupationNo)
	_Primary Key: memberNo, occupationNo
	_Foreign Key: memberNo 	references Member(memberNo)		NOT NULL	ON UPDATE CASCADE	ON DELETE CASCADE
	_Foreign Key: occupationNo 	references Occupation(occupationNo)		NOT NULL	ON UPDATE CASCADE	ON DELETE NO ACTION


IsInGroup (memberNo, socialGroupNo)
	_Primary Key: memberNo,  socialGroupNo
	_Foreign Key: memberNo 	references Member(memberNo)		NOT NULL	ON UPDATE CASCADE	ON DELETE CASCADE
	_Foreign Key: socialGroupNo references socialGroup(socialGroupNo)		NOT NULL	ON UPDATE CASCADE	ON DELETE NO ACTION


IsAbsentIn (memberNo, meetingNo, uponNotification, sameTimeOtherMeeting, absenceComments)
	_Primary Key: memberNo,  meetingNo
	_Foreign Key: memberNo 	references Member(memberNo)		NOT NULL	ON UPDATE CASCADE	ON DELETE CASCADE
	Foreign Key: meetingNo 	references Meeting( meetingNo)		NOT NULL	ON UPDATE CASCADE	ON DELETE NO ACTION
	Foreign Key:  sameTimeOtherMeeting  references Meeting(meetingNo)		ON UPDATE CASCADE	ON DELETE SET NULL


ChangedMembershipStatus (memberNo, meetingNo, dateOfChange, newStatus, reasonOfChange)
	_PrimaryKey: memberNo, dateOfChange, meetingNo
	_Foreign Key: memberNo references Member(memberNo)		NOT NULL	ON UPDATE CASCADE	ON DELETE CASCADE
	_Foreign Key: meetingNo references Meeting(meetingNo)		NOT NULL	ON UPDATE CASCADE	ON DELETE NO ACTION



/*  Multi-valued Attributes  */

MobilePhone (mobilePhoneNumber, memberNo)
	_Primary Key:  mobilePhoneNumber
	_Foreign Key: memberNo	references Member(memberNo)		NOT NULL	ON UPDATE CASCADE	ON DELETE CASCADE


DesktopPhone (desktopPhoneNumber, memberNo)
	_Primary Key:  desktopPhoneNumber
	_Foreign Key: memberNo	references Member(memberNo)		NOT NULL	ON UPDATE CASCADE	ON DELETE CASCADE

DecisionDocument (docNo, decDocument, meetingNo)
	_Primary Key: docNo
	_Foreign Key: meetingNo	references Meeting(meetingNo)		NOT NULL	ON UPDATE CASCADE	ON DELETE NO ACTION
