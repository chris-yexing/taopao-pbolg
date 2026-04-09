## ADDED Requirements

### Requirement: Article List Display
The system SHALL display a list of all articles on the blog page, sorted by update time in descending order, with pagination.

#### Scenario: Article list loads
- **WHEN** user visits the blog homepage or list page
- **THEN** system displays articles in reverse chronological order
- **AND** shows pagination controls if more than 10 articles

### Requirement: Individual Article Display
The system SHALL display full article content, including title, date, tags, and body.

#### Scenario: Article page loads
- **WHEN** user clicks on an article title
- **THEN** system shows the full article with metadata

### Requirement: Tag-based Filtering
The system SHALL allow filtering articles by tags, showing a list of articles for each tag.

#### Scenario: Tag filter applied
- **WHEN** user clicks on a tag
- **THEN** system displays only articles with that tag