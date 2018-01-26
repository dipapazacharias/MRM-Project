
	|===  Anasynthesi Project  ===|



------ PostgreSQL DB physical design ------



role: pioneer

DATABASE “anasynthesi_onra”


/* Data type definitions */
CREATE TYPE structure_type AS ENUM (‘base’, ‘theme’, ‘local’, ‘central’, ‘topcoord’);
CREATE TYPE structure_level AS ENUM (‘1’, ‘2’, ‘3’); /* 1-base, theme  2-local */
CREATE TYPE gender_type AS ENUM (‘F’, ‘M’, ‘T’);
CREATE TYPE membership AS ENUM (‘member’, ‘friend’, ‘semi_expelled’, ‘expelled’, ‘leaved’);
CREATE TYPE place_type AS ENUM (‘residence’, ‘origin’, ‘second_origin’);
CREATE TYPE payment_type AS ENUM (‘fee’, ‘donation’, ‘expense’);
CREATE TYPE pay_how AS ENUM (‘cash’, ‘paypal’);


/* Constraint definitions  */
/* CREATE CONSTRAINT  */


/* Domain definitions  */

CREATE DOMAIN SimpleDate	AS	date;
CREATE DOMAIN DateHour		AS	timestamp(0);
CREATE DOMAIN Comments		AS	text;

CREATE DOMAIN StructureNumber AS 	smallint;
CREATE DOMAIN StructureName 	AS	varchar(25);
CREATE DOMAIN StructureType		AS	structure_type;
CREATE DOMAIN StructureLevel	AS	structure_level;

CREATE DOMAIN MemberNumber		AS	integer;
CREATE DOMAIN MemberName	AS	varchar(35);
CREATE DOMAIN Email			AS	varchar(30);
CREATE DOMAIN Gender		AS	gender_type;
CREATE DOMAIN MembershipStatus	AS	membership;
CREATE DOMAIN Password			AS	varchar(20);
CREATE DOMAIN ViewLevel			AS	structure_level	DEFAULT ‘1’;
CREATE DOMAIN Absences			AS	smallint	DEFAULT 0;

CREATE DOMAIN OccupationNumber	AS	smallint;
CREATE DOMAIN OccupationName		AS	varchar(20);

CREATE DOMAIN SocialGroupNumber 	AS	smallint;
CREATE DOMAIN SocialGroupName	AS	varchar(20);

CREATE DOMAIN PlaceNumber	AS	smallint;
CREATE DOMAIN PlaceName		AS	varchar(20);
CREATE DOMAIN PlaceType		AS	place_type;
CREATE DOMAIN AddressRoad		AS	varchar(25);
CREATE DOMAIN AddressNumber	AS	varchar(7);

CREATE DOMAIN MeetingNumber	AS	integer;

CREATE DOMAIN PaymentNumber 	AS 	integer;
CREATE DOMAIN PaymentType	AS	payment_type;
CREATE DOMAIN Amount		AS	integer;
CREATE DOMAIN PayHow		AS	pay_how;

CREATE DOMAIN Notification	AS	boolean;

CREATE DOMAIN NewStatus			AS	membership;

CREATE DOMAIN MobilePhoneNumber 	AS	varchar(15);
CREATE DOMAIN DesktopPhoneNumber	AS	varchar(15);

CREATE DOMAIN DocNumber	AS	integer;
CREATE DOMAIN Document	AS	cidr;





/* Table definitions */

CREATE TABLE Structure(
	structureNo			StructureNumber		PRIMARY KEY,
	structureName		StructureName			UNIQUE NOT NULL,
	structureType		StructureType			NOT NULL,
	dateFounded			SimpleDate 				NOT NULL,
	dateDeprecated 	SimpleDate,
	directedBy 			StructureNumber,
FOREIGN KEY (directedBy) REFERENCES Structure(structureNo) ON UPDATE CASCADE	ON DELETE SET NULL
);


CREATE TABLE Member(
	memberNo			MemberNumber		PRIMARY KEY,
	firstName			MemberName			NOT NULL,
	lastName			MemberName			NOT NULL,
	email					Email						NOT NULL,
	dateOfBirth		SimpleDate			NOT NULL,
	gender				Gender					NOT NULL,
	membershipStatus	MembershipStatus	NOT NULL,
	comments		 	Comments,
	password		 	Password				NOT NULL,
	coordinator		StructureNumber,
	viewLevel			ViewLevel				NOT NULL,
	numberOfMeetingAbsences	 Absences		NOT NULL,
FOREIGN KEY (coordinator) REFERENCES Structure(structureNo) ON UPDATE CASCADE 	ON DELETE SET NULL
);


CREATE TABLE Occupation(
	occupationNo		OccupationNumber	PRIMARY KEY,
	occupationName	OccupationName		NOT NULL UNIQUE
);


CREATE TABLE SocialGroup(
	socialGroupNo	 	SocialGroupNumber	PRIMARY KEY,
	socialGroupName	SocialGroupName		NOT NULL UNIQUE
);


CREATE TABLE Country(
	countryNo		PlaceNumber 	PRIMARY KEY,
	countryName	PlaceName			NOT NULL UNIQUE
);


CREATE TABLE Area(
	areaNo			PlaceNumber	PRIMARY KEY,
	areaName		PlaceName		NOT NULL UNIQUE,
	countryNo		PlaceNumber	NOT NULL,
FOREIGN KEY (countryNo) REFERENCES Country(countryNo) ON UPDATE CASCADE 	ON DELETE RESTRICT
);


CREATE TABLE County(
	countyNo		PlaceNumber		PRIMARY KEY,
	countyName	PlaceName			NOT NULL UNIQUE,
	areaNo			PlaceNumber		NOT NULL,
FOREIGN KEY (areaNo) REFERENCES Area(areaNo) ON UPDATE CASCADE 	ON DELETE RESTRICT
);


CREATE TABLE Municipality(
	municipalityNo		PlaceNumber		PRIMARY KEY,
	municipalityName	PlaceName			NOT NULL,
	countyNo					PlaceNumber		NOT NULL,
FOREIGN KEY (countyNo) REFERENCES County(countyNo) ON UPDATE CASCADE 	ON DELETE RESTRICT
);


CREATE TABLE District(
	districtNo			PlaceNumber		PRIMARY KEY,
	districtName		PlaceName			NOT NULL,
	municipalityNo	PlaceNumber		NOT NULL,
FOREIGN KEY (municipalityNo) REFERENCES Municipality(municipalityNo) ON UPDATE CASCADE 	ON DELETE RESTRICT
);


CREATE TABLE Place(
	districtNo			PlaceNumber		NOT NULL,
	placeComments		Comments,
	placeType				PlaceType			NOT NULL,
	addressRoad			AddressRoad		NOT NULL,
	addressNumber	 	AddressNumber,
	memberNo				MemberNumber	NOT NULL,
PRIMARY KEY (memberNo, placeType),
FOREIGN KEY (memberNo) REFERENCES Member(memberNo) ON UPDATE CASCADE 	ON DELETE CASCADE,
FOREIGN KEY (districtNo) REFERENCES District(districtNo) ON UPDATE CASCADE ON DELETE RESTRICT
);


CREATE TABLE Meeting(
	meetingNo		MeetingNumber			PRIMARY KEY,
	structureNo	StructureNumber		NOT NULL,
	meetingDate	DateHour					NOT NULL,
FOREIGN KEY (structureNo) REFERENCES Structure(structureNo) ON UPDATE CASCADE  ON DELETE RESTRICT
);


CREATE TABLE Payment(
	paymentNo		PaymentNumber		PRIMARY KEY,
	memberNo		MemberNumber		NOT NULL,
	paymentDate	SimpleDate			NOT NULL,
	paymentType	PaymentType			NOT NULL,
	amount		Amount				NOT NULL,
	payHow		PayHow				NOT NULL,
	paymentReason	Comments,
FOREIGN KEY (memberNo) REFERENCES Member(memberNo) ON UPDATE CASCADE 	ON DELETE NO ACTION
);


CREATE TABLE HasMembers(
	memberNo		MemberNumber			NOT NULL,
	structureNo	StructureNumber		NOT NULL,
PRIMARY KEY(memberNo, structureNo),
FOREIGN KEY (memberNo) REFERENCES Member(memberNo) ON UPDATE CASCADE	ON DELETE CASCADE,
FOREIGN KEY (structureNo) REFERENCES Structure(structureNo) ON UPDATE CASCADE 	ON DELETE RESTRICT
);
/*  constraint needed: member cannot be to 2 different structures of same structureType  */


CREATE TABLE HasOccupation(
	memberNo			MemberNumber				NOT NULL,
	occupationNo	OccupationNumber		NOT NULL,
PRIMARY KEY (memberNo, occupationNo),
FOREIGN KEY (memberNo) REFERENCES Member(memberNo) ON UPDATE CASCADE	ON DELETE CASCADE,
FOREIGN KEY (occupationNo) REFERENCES Occupation(occupationNo) 	ON UPDATE CASCADE  ON DELETE RESTRICT
);
/*  constraint needed: up to 2 occupation registrations per member  */


CREATE TABLE IsInGroup(
	memberNo				MemberNumber				NOT NULL,
	socialGroupNo		SocialGroupNumber		NOT NULL,
PRIMARY KEY (memberNo, socialGroupNo),
FOREIGN KEY (memberNo) REFERENCES Member(memberNo) ON UPDATE CASCADE	ON DELETE CASCADE,
FOREIGN KEY (socialGroupNo) REFERENCES SocialGroup(socialGroupNo) ON UPDATE CASCADE	 ON DELETE RESTRICT
);
/*  constraint needed: up to 2 social group registrations per member  */


CREATE TABLE IsAbsentIn(
	memberNo			MemberNumber		NOT NULL,
	meetingNo			MeetingNumber		NOT NULL,
	uponNotification	 	Notification		NOT NULL,
	sameTimeOtherMeeting	MeetingNumber,
	absenceComments		Comments,
PRIMARY KEY (memberNo, meetingNo),
FOREIGN KEY (memberNo) REFERENCES Member(memberNo) ON UPDATE CASCADE	ON DELETE CASCADE,
FOREIGN KEY (meetingNo) REFERENCES Meeting (meetingNo) ON UPDATE CASCADE	ON DELETE RESTRICT,
FOREIGN KEY (sameTimeOtherMeeting) REFERENCES Meeting (meetingNo) 	ON UPDATE CASCADE ON DELETE RESTRICT
);


CREATE TABLE ChangedMembershipStatus(
	memberNo		MemberNumber		NOT NULL,
	meetingNo		MeetingNumber		NOT NULL,
	dateOfChange	SimpleDate		NOT NULL,
	newStatus		NewStatus				NOT NULL,
	reasonOfChange	Comments		NOT NULL,
PRIMARY KEY (memberNo, dateOfChange, meetingNo),
FOREIGN KEY (memberNo) REFERENCES Member(memberNo) ON UPDATE CASCADE	ON DELETE CASCADE,
FOREIGN KEY (meetingNo) REFERENCES Meeting(meetingNo) ON UPDATE CASCADE ON DELETE RESTRICT
);


CREATE TABLE MobilePhones(
	mobilePhoneNumber	MobilePhoneNumber	PRIMARY KEY,
	memberNo					MemberNumber			NOT NULL,
FOREIGN KEY (memberNo) REFERENCES Member(memberNo) ON UPDATE CASCADE	ON DELETE CASCADE
);
/*  Constraint needed: up to 2 phone numbers per member  */


CREATE TABLE DesktopPhones(
	desktopPhoneNumber	DesktopPhoneNumber	PRIMARY KEY,
	memberNo						MemberNumber				NOT NULL,
FOREIGN KEY (memberNo) REFERENCES Member(memberNo) ON UPDATE CASCADE	ON DELETE CASCADE
);
/*  Constraint needed: up to 2 phone numbers per member  */


CREATE TABLE DecisionDocuments(
	docNo	 			DocNumber		PRIMARY KEY,
	decDocument	Document		NOT NULL,
	meetingNo		MeetingNumber	NOT NULL,
FOREIGN KEY (meetingNo) REFERENCES Meeting(meetingNo) ON UPDATE CASCADE ON DELETE RESTRICT
);





/*

	Derived Attributes
		/structureLevel
		/age  =  CURRENT DATE – dob,
		/numberOfMeetingAbsences  =  SUM(IsAbsentIn WHERE memberNo = memberNo,
		/viewLevel  =  FROM Has WHERE EXISTS structureNo FROM Sructure WHERE structureType = A,B,C,D )
		/absenceComments ??

*/