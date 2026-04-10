  

[[Lam dong trail]]

[[NutiZen]]

[[Nutizen thượng phẩm]]

[[KudoOne]]

# FastPass

```SQL
insert into participants(event_id,participant_id,info,send_mail_status,ticket_type,parent_participant_id) SELECT p.event_id,Concat(ap.participant_id, '_', ap.id),ap.info,'0','2',ap.participant_id FROM associate_participants ap join participants p on ap.participant_id=p.participant_id;
```

[[KudoFoto]]