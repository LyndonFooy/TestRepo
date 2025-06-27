# View: dbo.EmailQueue

## Definition

```sql

--Update the view that gets the data.
CREATE VIEW [dbo].[EmailQueue]
AS
	
SELECT
	lr.ContactId,
	lc.ListCampaignId,
	[LastSentOn],
	c.Title,
	c.Name,
	c.Surname,
	c.Mobile,
	c.Phone,
	c.Email AS EmailAddress,
	c.Custom1,
	c.Custom2,
	c.Custom3,
	c.Custom4,
	c.Custom5,
	c.ContactStatusEnum AS [Status],
	lr.Retries,
	lr.CreatedOn,
	lc.[Subject],
	'' AS FromAddress,
	'' AS FromName,
	lc.ContentUrl AS MessageUrl,
	'' AS ReplyAddress,
	0 AS [Priority]
	,[IsBouncer] = CASE WHEN (SELECT COUNT(*) FROM [BounceEmail] BE WHERE BE.[EmailAddress] = c.Email AND BE.[BounceType] = 'Transient' ) > 2 THEN 1
					WHEN (SELECT COUNT(*) FROM [BounceEmail] BE WHERE BE.[EmailAddress] = c.Email AND BE.[BounceType] <> 'Transient' ) > 0 THEN 1
						ELSE 0
					END
	, [IsComplainer] = CASE WHEN (SELECT COUNT(*) FROM [ComplaintEmail] CE WHERE CE.[EmailAddress] = c.Email) > 2 THEN 1
						ELSE 0
					END
	, [IsQuarantined] = CASE WHEN (SELECT COUNT(*) FROM [Contact] C WHERE C.[Email] = c.Email AND C.[ContactStatusEnum] = 'Quarantined') > 0 THEN 1
						ELSE 0
					END

	FROM [ListCampaignRecipient] lr
		JOIN [ListCampaign] lc ON lc.[ListCampaignId] = lr.[ListCampaignId]
		JOIN Contact c ON lr.ContactId = c.ContactId
	WHERE
		(lr.[CampaignContactStatusEnum] = 'Queued' OR lr.[CampaignContactStatusEnum] = 'Pending')
		AND ( lr.[LastSentOn] IS NULL OR lr.[LastSentOn] < GETDATE())


```
