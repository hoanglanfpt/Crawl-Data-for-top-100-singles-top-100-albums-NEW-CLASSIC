# Crawl-Data-for-top-100-singles-top-100-albums-NEW-CLASSIC

*Crawl Data của TopSingle, TopAlbum, NewClassic:*

988CE4D571B4455EA4B9B1904BA92916 - Priority 2019
E4B85D0A993146EEB84426C2246EFCA0 - Priority 2018
90ECECF350D94F8C8A16B209CADF9B9E - Priority 2017
Mẫu query insert của e:

INSERT INTO crawlingtasks(Id,Priority,ActionID,Taskdetail) VALUES (uuid4(),2019,'988CE4D571B4455EA4B9B1904BA92916','{"PIC":"Minchan"}');
Câu lệnh xuất của em:
Top100Single
SELECT
    ts.CreatedAt,
    ts.Genre,
    ts.Rank,
    ts.Ext->>'$.album_uuid' AS album_uuid,
    ts.Ext->>'$.album_title' AS album_title,
    ts.Ext->>'$.album_artist' AS album_artist,
    ts.Ext->>'$.album_url' AS album_url,
    ts.ArtistName,
    ts.ArtistUUID,
    ts.TrackTitle,
    ts.TrackId,
    ts.DataSourcesExisted->>'$.MP3_FULL' AS MP3_dsid,
    ts.DataSourcesExisted->>'$.MP4_FULL' AS MP4_dsid,
    ds.SourceURI AS MP3link, -- to change format
    ts.Ext->>'$.is_duplicate_track' AS Duplicate_Track,
    ts.Ext->>'$.need_checking_mp3' AS Checking_mp3,
    ts.Ext->>'$.need_checking_mp4' AS Checking_mp4,
    ts.Ext->>'$.already_existed_in' AS Already_existed
FROM
    reportautocrawler_topsingle ts
LEFT JOIN
  datasources ds
ON
  ds.id = ts.DataSourcesExisted ->> '$.MP3_FULL' -- to change format
WHERE
    ts.CreatedAt >= '2020-01-03 00:00:00' -- to change date
ORDER BY
    ts.Genre, ts.Rank ASC, ts.CreatedAt ASC
;
=> Xuất sheet MP4, sheet MP3 vào cùng 1 sheet
-------------------------------------------------------------------------
NewClassic
:one: Xuất list Albums của NewRelease
SELECT
DISTINCT
    nc.AlbumInfo->>'$.release_date' Release_date,
    Category,
    Source,
        nc.AlbumInfo->>'$.album_title' AS AlbumTitle,
        nc.AlbumInfo->>'$.album_artist' AS AlbumArtist,
        nc.AlbumInfo->>'$.album_url' AS AlbumURL,
				nc.TrackId AS _
FROM
    reportautocrawler_newclassic nc
LEFT JOIN
  datasources ds
ON
  ds.id = nc.DataSourcesExisted ->> '$.MP4_FULL'
WHERE
    nc.CreatedAt >= '2020-01-03 00:00:00'
		and nc.TrackId is null
ORDER BY
        CONCAT(nc.CreatedAt,nc.AlbumInfo->>'$.track_index') ASC
=> Sheet này tên là S11 ạ, ngoài ra, khi a xuất thì số lượng sẽ luôn > 70 albums a nhé
Sau khi bạn intern check xong mục :one: thì a đặt lệnh crawl giúp em ạ,
Mẫu query:
INSERT INTO crawlingtasks(Id,Priority,ActionID,Taskdetail) VALUES (uuid4(),2000,'9C8473C36E57472281A1C7936108FC06','{"album_id":"1492785997","is_new_release": true}');
Sau khi crawl xong thì sang bước 2
:two: Xuất MP4, MP3 để check
SELECT
    nc.CreatedAt,
    nc.AlbumInfo->>'$.release_date' Release_date,
    Category,
    Source,
		nc.Ext->>'$.album_uuid' AS Albumuuid,
        nc.AlbumInfo->>'$.album_title' AS AlbumTitle,
        nc.AlbumInfo->>'$.album_artist' AS AlbumArtist,
        nc.AlbumInfo->>'$.album_url' AS AlbumURL,
        nc.AlbumInfo->>'$.track_index' AS TrackNum,
    nc.ArtistName,
    ArtistUUID,
    TrackTitle,
    nc.TrackId,
    nc.Ext ->> '$.track_id' Subgerned,
    nc.DataSourcesExisted ->> '$.MP3_FULL' AS MP3_dsid,
    nc.DataSourcesExisted ->> '$.MP4_FULL' AS MP4_dsid,
    ds.SourceURI AS MP4link, -- to change format
		nc.Ext->>'$.need_subgenre' AS need_subgenre,
		nc.Ext->>'$.is_duplicate_track' AS Duplicate_Track,
    nc.Ext->>'$.need_checking_mp3' AS Checking_mp3,
    nc.Ext->>'$.need_checking_mp4' AS Checking_mp4,
    nc.Ext->>'$.already_existed_in' AS Already_existed
FROM
    reportautocrawler_newclassic nc
LEFT JOIN
  datasources ds
ON
  ds.id = nc.DataSourcesExisted ->> '$.MP4_FULL' -- to change format
WHERE
    nc.CreatedAt >= '2020-01-03 00:00:00' -- to change date
		and nc.TrackId is not null
ORDER BY
		CONCAT(nc.CreatedAt,nc.AlbumInfo->>'$.track_index') ASC
=> Xuất sheet MP4, MP3 để check
-------------------------------------------------------------------------
Top 100 Albums
SELECT
    ta.CreatedAt,
    ta.Genre,
    ta.Rank, 
    ta.AlbumTitle,
    ta.ItunesAlbumId,
    ta.Ext ->> '$.album_url' Ituneslink,
    ta.Ext ->> '$.album_uuid' AlbumUUID,
    ta.TrackNum,
    ta.Verification,        
    ta.ArtistName,
    ta.ArtistUUID,
    ta.TrackTitle,
    ta.TrackId,
    ta.DataSourcesExisted ->> '$.MP3_FULL' AS MP3_dsid,
    ta.DataSourcesExisted ->> '$.MP4_FULL' AS MP4_dsid,
    ds.SourceURI AS MP3link, -- to change format
    ds.Valid,
		ta.Ext->>'$.is_duplicate_album' AS Duplicate_Album,
    ta.Ext->>'$.is_duplicate_track' AS Duplicate_Track,
    ta.Ext->>'$.need_checking_mp3' AS Checking_mp3,
    ta.Ext->>'$.need_checking_mp4' AS Checking_mp4,
    ta.Ext->>'$.already_existed_in' AS Already_existed
FROM
    reportautocrawler_top100albums ta
LEFT JOIN
  datasources ds
ON
  ds.id = ta.DataSourcesExisted ->> '$.MP3_FULL' -- to change format
WHERE
    ta.CreatedAt >= '2020-01-03 00:00:00' -- to change date
ORDER BY
    ta.Genre, ta.Rank ASC, ta.TrackNum ASC, ta.CreatedAt ASC
=> Xuất sheet MP4, sheet MP3 vào cùng 1 sheet
