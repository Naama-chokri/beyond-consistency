# Prompts

This appendix gives the prompts used by our approach, exactly as sent to
every model. Words in braces are filled in for each claim. For the
argument generation and the relation audit, the system message is: “You
are an expert in claim verification, formal argumentation and strict
JSON output.” The prompts of the ArgLLM baseline are those of the
authors’ repository .

## Fact generation

System message: “You output atomic factual building blocks. You never
conclude.” Temperature 0.3. Used on COM2SENSE, $\alpha$NLI, and
CosmosQA; on StrategyClaim the dataset’s facts are used instead.

    You are building atomic factual building blocks,
    in the style of a reasoning dataset.

    For the claim below, produce 3-5 SHORT, SEPARATE
    factual statements. Each must be:
    - A single atomic fact about ONE concept in the
      claim.
    - General commonsense knowledge, stated on its
      own.
    - NOT a comparison, ranking, or conclusion.
    - NOT a statement about whether the claim is true
      or false.

    Claim: "{claim}"

    Return 3-5 atomic facts, one per line. Do not
    conclude.

## Argument generation and self-scoring

Temperature 0.15. The output is parsed as JSON.

    Construct candidate arguments for a Bipolar
    Argumentation Framework used for claim
    verification.

    CLAIM:
    {claim}

    AVAILABLE FACTS:
    {facts}

    First determine the intended meaning of the claim,
    then reason carefully from facts and commonsense.

    Generate up to 5 PRO candidates and up to 5 CON
    candidates.

    PRO: a specific, grounded reason the claim is
    TRUE.
    CON: a specific, grounded reason the claim is
    FALSE.

    CRITICAL rules for CON arguments (attacks):
    - Only generate a CON if there is a GENUINE,
      SPECIFIC, STRONG reason the claim is false.
    - Do NOT generate speculative CONs like "maybe not
      always", "it might not", "in some cases".
    - Do NOT generate a CON that merely raises a weak
      possibility; it must give a real reason the
      claim is false.
    - If the claim is clearly true and there is no
      strong objection, generate ZERO CON arguments.

    CRITICAL rules for PRO arguments:
    - Only generate a PRO if there is genuine support.
      If the claim is clearly false, generate ZERO PRO
      arguments.
    - Do NOT generate circular or vague arguments.

    General:
    - Each argument must be specific and grounded,
      under 30 words.
    - Do not output the final TRUE/FALSE label.

    Score each candidate 0 to 1 for: relevance;
    grounding; clarity; overall_quality.
    Score speculative, weak, vague, or circular
    arguments BELOW 0.4 on overall_quality.

    Return valid JSON only:
    {
      "interpreted_claim": "brief meaning",
      "pro": [{"text":"...","relevance":0.0,"grounding
      ":0.0,"clarity":0.0,"overall_quality":0.0}],
      "con": [{"text":"...","relevance":0.0,"grounding
      ":0.0,"clarity":0.0,"overall_quality":0.0}]
    }

## Relation auditing

Temperature 0. Candidate relations are sent in batches of 16; the output
is parsed as JSON.

    Audit candidate relations for a Bipolar
    Argumentation Framework.

    CLAIM:
    {claim}
    FACTS:
    {facts}
    ARGUMENTS:
    {arguments}

    For every candidate, choose exactly one allowed
    type.
    SUPPORT: source gives a meaningful direct reason
    to accept target.
    ATTACK: source directly contradicts, rebuts or
    undermines target.
    NONE: no meaningful direct relation.
    Use ATTACK for CON->CLAIM only if the CON gives a
    genuine reason the claim is false or doubtful.
    Use NONE when the CON concedes the claim or
    agrees.
    Keep every reason at eight words or fewer.
    For ATTACK/SUPPORT scores normally exceed 0.50;
    for NONE normally below 0.50.

    CANDIDATES:
    {candidates}

    Return valid JSON only:
    {"relations":[{"source":"P1","target":"CLAIM",
      "type":"SUPPORT","target_match":0.9,
      "grounding":0.9,"relation_score":0.9,
      "reason":"..."}]}
