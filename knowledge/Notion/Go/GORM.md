  

# Self reference

```Go
type Participant struct {
	Model

	EventID            ID
	ParticipantID      string
	CheckedInAt        *time.Time
	CheckedInBy        string
	PickedRaceKitAt    *time.Time
	PickedRaceKitBy    string
	SignatureImageUrls StringArray
	CheckinImageUrls   StringArray
	IdppImageUrls      StringArray
	Info               JSON
	IsAuthorized       bool
	IsAssociated       bool
	IsNeedGuardianInfo bool
	GuardianInfo       *ParticipantGuardianInfo
	SendMailStatus     SendMailStatus
	TicketType         TicketType

	ParentParticipantID *string

	AssociateParticipants []Participant              `gorm:"foreignKey:ParentParticipantID;references:ParticipantID"`
	AuthorizedData        *ParticipantAuthorizedData `gorm:"foreignKey:ParticipantID;references:ParticipantID"`
	ModifiedAt            time.Time
}
```

  

```Go
AssociateParticipants []Participant  `gorm:"foreignKey:ParentParticipantID;references:ParentParticipantID"`
```

  
có participantID nhưng ko có parent  

```Go
AssociateParticipants []Participant `gorm:"references:ParentParticipantID"`
```

có parent participant nhưng ko có ParticipantID

  

Solution

```Go
AssociateParticipants []Participant `gorm:"foreignKey:ParentParticipantID;references:ParticipantID"`
```

  

## Conclusion

gorm:"foreignKey:ParentParticipantID;references:ParticipantID"

a field has to define foreignKey(parent field) and references(child field).

Nếu chỉ define một tag GORM sẽ set vào giá trị bị thiếu value đó.

  

```Go
// AttQuestion mapped from table <attendant_questions>
type AttQuestion struct {
	ID          int32                 `gorm:"column:id;primaryKey" json:"id"`
	Label       custom_type.MultiLang `gorm:"column:label;comment:show for user, multi language" json:"label"`                                                 // show for user, multi language
	Placeholder custom_type.MultiLang `gorm:"column:placeholder;comment:place holder, multi language" json:"placeholder"`                                      // place holder, multi language
	Validation  string                `gorm:"column:validation;comment:validate value using for both front and back end, ref go validation" json:"validation"` // validate value using for both front and back end, ref go validation
	Key         string                `gorm:"column:key;comment:the name of variable" json:"key"`                                                              // the name of variable
	Type        string                `gorm:"column:type;comment:data type, one of (group, text, number, select, date, phone...)" json:"type"`                 // data type, one of (group, text, number, select, date, phone...)
	Extends     string                `gorm:"column:extends" json:"extends"`
	ChildOf     *int32                `gorm:"column:child_of" json:"child_of"`

	// parent categories
	ChildOfModel *AttQuestion `gorm:"foreignKey:ChildOf;references:id"`
}
```