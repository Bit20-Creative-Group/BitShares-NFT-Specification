# A Specification for Non-Fungible Tokens (NFTs) on the BitShares Blockchain

What follows is a DRAFT proposed specification for standardizing or regularizing non-fungible tokens (NFTs) on the BitShares blockchain and similar blockchains.  It is a work in progress, and comments and suggestions are welcomed and encouraged.

## What makes an NFT

* Singular or very small integral issuance:
  * Asset precision should be zero
  * Asset should have a max_supply of 1 or of a very small limited number
* Immutability
  * BitShares assets have a disavowable permission to increase max supply. This permission must be disavowed by the issuer, else there remains a mechanism to increase issuance.
  * Asset descriptions — important metadata that uniquely identifies the digital entity encapsulated by the NFT is stored in the Asset Description field.  Since this field is mutable, the possibility exists for the asset issuer to alter, deface, spoil, or otherwise destroy the NFT.  At present, there is no disavowable asset permission for editing the description.  For obvious reasons, it would be useful if there was one, to safeguard the NFT data, however there is not.  An alternative, however, once the NFT is created, is to transfer asset issuership of the NFT to `null-account`, whereby future asset update operations will become impossible.

## Asset Description Langauge

The asset `description` field can accommodate arbitrary text.  Current convention is to store a JSON blob allowing asset issuers to embed three distinct descriptive properties of the asset.  This JSON blob has the following form:  (whitespace added for visual clarity)

```
{
  "main": "A magical asset that you will surely want to HODL",
  "market": "BTS",
  "short_name": "Magical Asset"
}
```

Where the fields have the following meanings:

| | |
|--------|---------|
| `main` | Main asset description. Asset issuers can write basically whatever they want. |
| `market` | "SYMBOL" — Client software uses this symbol to indicate a preferred, suggested, or default market this asset should trade against. |
| `short_name` | A short (max 32 chars) name of the asset.  (E.g. "Bitcoin", for asset BTC.) |
| | |

To create a NFT, this document proposes the addition of two new fields to this JSON blob.   The keys are `nft_object` and `nft_signature`, so that the blob will now have the following form:

```
{
  "main": "A magical asset that you will surely want to HODL",
  "market": "BTS",
  "nft_object": { ... },
  "nft_signature": "20ff....ea",
  "short_name": "Magical Asset"
}
```

with the following meanings:

| | |
|--------|---------|
| `nft_object` | A JSON object containing NFT content and attributes, in a format and containing fields as described below |
| `nft_signature` | A signature (ECDSA or similar) from a well-known public key of the artist. The artist signs the ascii serialization of the contents of the `nft_object` to obtain the signature. Client software can verify that the signature is valid. |
| | |

## NFT Object

The NFT object shall be represented as a canonicalized JSON blob containing all the of required fields and some combination of the optional fields described below.  A canonicalized JSON blob is an ascii JSON serialization in which keys appear in lexicographically sorted order, and in which all non-quoted whitespace is removed (no space between keys and values, no linebreaks, no indentation, etc.). Additionally, quoted text should escape control sequences and special characters (such as linebreaks, tabs, unicode characters, etc.).

### Required keys:

The following keys are required of all NFTs. Following that, some keys will be required depending on the type of NFT defined.

| | |
|-|-|
| `type` | Should be one of:<br><br>&nbsp;&nbsp;"NFT/ART",<br>&nbsp;&nbsp;"NFT/DOCUMENT",...<br><br>(Others t.b.d.) Types can optionally be subspaced to convey extra context by appending "/SUBSPACE". (Possible examples: "NFT/ART/VISUAL", "NFT/ART/MUSIC".) |
| `attestation` | Here the artist commits or dedicates the artwork to the blockchain, expressly naming the token or asset ID under which the work will live, and attests to its qualified or unqualified uniqueness. E.g., an artist may attest that no other NFT encapsulation of this work exists, and may declare intent as to future re-issues, or tokenization on other chains, etc.<br><br>Example: _"I, [Artist Name], originator of the work herein, hereby commit this piece of art to the BitShares blockchain, to live as the token named TOKEN.NAME, and attest that no prior tokenization of this art exists or has been authorized by me. The work is original, and is fully mine to dedicate in this way. The right to re-issue this artwork under other tokens is [reserved, disavowed]."_ |
| `encoding` | Typically "base64", and indicates that the binary data of the media item or other binary fields have been serialized to ascii using base64 encoding |
| `pubkeyhex` | Hex encoding of the bytes of the artist's public key in compressed form.  This will be used to validate the artist's signature. (NOTE: While this allows to validate the signature, it does not _authenticate_ the signature.  Establishing whether this is in fact the public key of the artist is a separate process.) |
| | |

#### Type spaces:

| | |
|-|-|
| `NFT/ART` | Artistic works |
| `NFT/DOCUMENT` | Example: a journalist writes an article, or an author writes a short story, and the token conveys publication rights. |
| | |

#### Keys required by type NFT/ART:

The following keys are considered required for the NFT to be formally correct.  (Technically, this is more of a "strong suggestion".)  However, fields may be empty strings if the artist does not wish to supply content for them.

| | |
|-|-|
| `title` | Title of the work |
| `artist` | Name or pseudonym of the artist. May also include aliases or online names or handles, to include blockchain account names or addresses which might facilitate authenticating a signing key. Example: "Arty McArtface (on BitShares as @artface)" |
| `narrative` | A personal statement from the artist describing the work, such as what the work means to them, or what inspired it.  May include details of it's creation, etc.  It's a freeform field, and can be adapted as appropriate for the piece.  Example, if the work is an avatar, playing card, role playing character, etc., then this field may also include stats and abilities, strengths, weaknesses, etc.  |
| | |


#### Media item keys for type NFT/ART:

ONE of the following keys must be included to embed the media item if the value of the `type` field is "NFT/ART".  The particular key used indicates the file type.  Note, the data contained in the value is encoded according to the value of the required `encoding` key, unless the key is a multihash key, in which case the `encoding` key can be ignored.  This is not a comprehensive list.  

| | |
|-|-|
| `image_png` | An image file in PNG format |
| `image_gif` | An image file in GIF format |
| `image_jpeg` | An image file in JPEG format |
| `image_png_multihash` | An ipfs multihash of an image file in PNG format |
| `image_gif_multihash` | An ipfs multihash of an image file in GIF format |
| `image_jpeg_multihash` | An ipfs multihash of an image file in JPEG format |
| | |

### Optional Keys:

| | |
|-|-|
| `tags` | Comma-separated list of keywords to facilitate topic/interest searches. |
| `flags` | Comma-separated list of semistandardized FLAG keywords to indicate important information to viewers and parsers. Example: "NSFW" | 
| `password_multihash` | If the media item and/or narrative fields are encrypted, e.g. with AES encryption, this field can contain the ipfs multihash of a file containing either the unlock passphrase or other instructions for how to decrypt the work. (Note that ipfs multihashes can be computed *without* necessarily publishing, so that this multihash provides a mechanism to reveal the decryption keys at a future date, and publish in such a way that an NFT viewer can easily retrieve the needed information for rendering. A standardized format for these password files is T.B.D.) |
| `license` | If the token grants additional license for the use of the creative work outside of the usual and customary rendering or presentation by NFT viewers, wallets, block explorers, decentralized exchanges, or the sharing of such representations on social media or other venues, or other activities or utilization generally accompanying or associated with NFT community and culture, such grants of license may be articulated here.  Note that this is an OPTIONAL field meant to extend the utility of a token by including grants of license not considered implicitly granted by the creation of the NFT itself. It is NOT meant as a way to place restrictions that would go against what is considered regular and customary for NFTs.  An example of such a grant of license might be granting to the holder of the NFT token the right print and sell physical copies of the tokenized artwork, or to collect royalties for commercial use of the artwork, etc. |
| | |

## Discussion

Discussion can occur here: [Issues](../../issues).

### Strategies for reducing resource utilization by NFTs

NFTs impose a RAM burden on witness and API nodes by virtue of the fact that asset objects, including description field are kept in RAM. Token creators should do their utmost to reduce the storage footprints of their artwork, however, a storage burden in some form is inescapable. Aside from storing links to content instead of actual content data, one option to reduce the RAM hit might be to publish the NFT with full artwork, but then in a subsequent block to replace the nft_object field contents and replace with an object that refers to the block number in which the original publication occurred.  Further discussion of this possibility is [here: (issues/1)](../../issues/1).
