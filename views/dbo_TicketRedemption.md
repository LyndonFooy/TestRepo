# View: dbo.TicketRedemption

## Definition

```sql
CREATE VIEW [dbo].[TicketRedemption] 
AS

SELECT
	[RedemptionData].*,
	(SELECT TOP 1 RedemptionDeviceID FROM dbo.RedemptionDevice WHERE EventId = [RedemptionData].EventId)  AS [RedemptionDeviceId],
	(SELECT TOP 1 [Label] FROM dbo.RedemptionDevice WHERE EventId = [RedemptionData].EventId)  AS [DeviceLabel],
	[RedemptionData].[LastModifiedOn] AS [RedeemedOn]
FROM
	(
	SELECT
		[Bookings].*,
		[Seating].[Type] [SeatingType],
		[Seating].[GateNumber]
	FROM
		(
			SELECT
				E.[EventId],
				E.[ParentEventId],
				E.[Name] [EventName],
				B.[BookingId],
				B.[BookingDate],
				B.[LastModifiedOn],
				CASE WHEN CHARINDEX('/',B.[Ref]) > 0 THEN (LTRIM(SUBSTRING(B.[Ref], 0,CHARINDEX('/',B.[Ref])))) ELSE B.[Ref] END [BookingReference],
				CASE WHEN CHARINDEX('/',B.[Ref]) > 0 THEN (LTRIM(SUBSTRING(B.[Ref], CHARINDEX( '/',B.[Ref])+1,len(B.[Ref])+1))) ELSE '' END [IDNumber],				
				B.[Status] [BookingStatus],
				B.[TicketId],
				T.[Description] [TicketDescription],
				T.[BasePrice] [TicketPrice],
				TRX.[TransactionId],
				TRX.[Ref] [TransactionReference],
				TRX.[Status] [TransactionStatus],
				TRX.[Tag],
				[BookingCustomer].[CustomerId],
				[BookingCustomer].[Title],
				[BookingCustomer].[Gender],
				[BookingCustomer].[CustomerName],
				[BookingCustomer].[CustomerSurname],
				[BookingCustomer].[Email],
				[BookingCustomer].[Mobile],
				[BookingCustomer].[Phone],
				[BookingCustomer].[Birthday],
				[BookingSeat].[SeatId],
				[BookingSeat].[SeatNumber],
				[BookingSeat].[SeatRowLabel],
				[BookingSeat].[SeatRowNumber],
				[BookingSeat].[SeatGroup],
				[BookingSeat].[SeatingBlockId],
				[BookingSeat].[SeatingBlockTitle],
				[UserTransaction].[PaymentType],
				[POSUser].[UserName]
			FROM [Event] E
				JOIN [Ticket] T ON E.[EventId] = T.[EventId]
				JOIN [Booking] B ON T.[TicketId] = B.[TicketId]
				JOIN
				(
					SELECT
						B.[BookingId],
						S.[SeatId],
						S.[RowLabel] [SeatRowLabel],
						S.[RowNumber] [SeatRowNumber],
						S.[SeatNumber],
						S.[Group] [SeatGroup],
						SB.[SeatingBlockId],
						SB.[Title] [SeatingBlockTitle],
						SB.[GateNumber]
					FROM [Booking] B
						LEFT OUTER JOIN [Seat] S ON B.[SeatId] = S.[SeatId]
						LEFT OUTER JOIN [SeatingBlock] SB ON S.[SeatingBlockId] = SB.[SeatingBlockId]
				) [BookingSeat] ON B.[BookingId] = [BookingSeat].[BookingId]
				JOIN
				(
					SELECT
						B.[BookingId],
						C.[CustomerId],
						C.[Name] [CustomerName],
						C.[Surname] [CustomerSurname],
						C.[Email],
						C.[Phone],
						C.[Mobile],
						C.[Title],
						C.[Gender],
						C.[Birthday],
						B.[TransactionId]
					FROM [Booking] B
						LEFT OUTER JOIN [Customer] C ON B.[CustomerId] = C.[CustomerId]
				) [BookingCustomer] ON [BookingSeat].[BookingId] = [BookingCustomer].[BookingId]
				JOIN [Transaction] TRX ON [BookingCustomer].[TransactionId] = TRX.[TransactionId]
				LEFT OUTER JOIN (
					SELECT [UPT].[UserId],[UPT].[TransactionId],[UPT].[PaymentType]
					FROM [UserPosTransaction] AS UPT
					) AS [UserTransaction] ON TRX.[TransactionId] = [UserTransaction].[TransactionId]
				LEFT OUTER JOIN (
					SELECT [UP].[UserId], [UP].[UserName]
					FROM [UserProfile] AS UP
					) AS [POSUser] ON [UserTransaction].[UserId] = [POSUser].[UserId]
		) [Bookings]
		LEFT OUTER JOIN
		(
			SELECT
				SB.[SeatingBlockId],
				SB.[GateNumber],
				ESA.[TicketId],
				ESA.[Type]
			FROM [EventSeatingArrangement] ESA
				JOIN [SeatingBlock] SB ON ESA.[SeatingBlockId] = SB.[SeatingBlockId]
		) [Seating] ON [Bookings].[TicketId] = [Seating].[TicketId] AND [Bookings].[SeatingBlockId] = [Seating].[SeatingBlockId]
	) [RedemptionData]
	--LEFT OUTER JOIN
	--(
	--	SELECT 
	--		RD.[RedemptionDeviceId],
	--		RD.[Label] [DeviceLabel],
	--		RDL.[CreatedOn] [CreatedOn],
	--		RDL.[BookingRef]
	--	FROM [RedemptionDevice] RD
	--		JOIN
	--		(
	--			SELECT 
	--				*, 
	--				ROW_NUMBER() OVER(PARTITION BY [BookingRef] ORDER BY [CreatedOn]) AS RowNum
	--			FROM [RedemptionDeviceLog]
	--		) RDL ON RD.[RedemptionDeviceId] = RDL.[RedemptionDeviceId]
	--	WHERE RDL.[RowNum] = 1
	--) RDL ON [RedemptionData].[BookingReference] = RDL.[BookingRef]
	WHERE [RedemptionData].[TransactionStatus] = 'Settled'


```
