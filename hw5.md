# HW5 - Database design

## Part 1 - Design the Database
'''sql
create table plans (
  id int primary key generated always as identity,
  name text not null,
  created_at timestamptz default now ()
);

create table interviews (
  id int primary key generated always as identity,
  created_at timestamptz default now(),
  interviewee text,
  interview_date date,
  content text, 
  plan_id int not null references plans (id)
); 

create table research_questions (
  id int primary key generated always as identity,
  content text not null,
  created_at timestamptz default now(),
  interview_id int not null references interviews (id)
);

create table interview_questions (
  id int primary key generated always as identity, 
  content text not null,
  created_at timestamptz default now(),
  research_id int not null references research_questions (id)
);
'''

## Part 2 - Populate Database

insert into plans (name) values 
  ('Mobile new onboarding study'),
  ('Web-app report v.2 study');
  
insert into interviews (interviewee, interview_date, content, plan_id) values 
  ('Nghi Nguyen', '2026-09-20', 'My team and I created Bird DJ, a vibecoded soundboard that would facilitate playing different bird calls and songs, bird descriptions, field notes, and more.', 1),
  ('Ngoc Nguyen', '2026-09-22',' We asked participants to design a retrospective birding audio experience that would showcase their latest birding encounter.', 2);

insert into research_questions (content, interview_id) values
  ('Do user feel it easy to use?', 1),
  ('Do they remember the values it bring?',1);

insert into interview_questions (content, research_id) values 
  ('Can they complete OBD quickly?',1),
  ('Did they confuse when finish the job?',1);

## Part 3 / Iteration 1 - Table changes

### Change the database

alter table interviews 
  add column status text
  check (status in ('planned','completed','cancelled'));

### Update existing row

update interviews
  set status ='planned'
  where id = 1;

update interviews
  set status = 'completed'
  where id = 2;
select id, status from interviews;

### Show planned interviews

select 
  interviews.interviewee, 
  interviews.status, 
  plans.name as plan_name
from interviews
join plans 
  on interviews.plan_id = plans.id
where interviews.status = 'planned';

### Show recently interview

select interviewee, created_at
from interviews
order by created_at desc
limit 1;

/* Iteration 1 - reflect: 
  If we drop & recreate a table, we will lost all data that we had (it's dangerous if we had real users) So we only modify the existing data table keep the data we had. */ 

## Part 4 / Iteration 2 : New Table

### Create new table
  
create table highlights (
  id int primary key generated always as identity,
  quote text not null,
  theme text 
    check (theme in('pain point', 'motivation', 'workaround')),
  created_at timestamptz default now(),
  interview_id int not null references interviews (id)
);

### Populate 

insert into highlights (quote, theme, interview_id) values
  ('For this project, we wanted to design something for not only a range of birding experiences but also a range of visual abilities.', 'motivation',1),
  ('We asked participants to design a retrospective birding audio experience that would showcase their latest birding encounter.', 'workaround',2),
  ('I had to sort all of the reviews based on the themes created by user interviews, and I only had a week to do it in.', 'pain point',1);

### Show 'pain point' hightlight

select 
  highlights.quote,
  highlights.theme,
  interviews.interviewee as interviewee
from highlights
join interviews
  on highlights.interview_id = interviews.id
where highlights.theme = 'pain point';

### Count

select theme, count(*)
from highlights
group by theme;

/* iteration 2 - reflect 
  Khi mà đối tượng có mối quan hệ 1-n thì ta nên tạo bảng mới để dễ quản lý, còn khi đối tượng có mối quan hệ 1-1 thì chỉ cần tạo thêm một cột trong bảng cũ. */
