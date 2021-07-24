# default parameters, which should not be changed.
TOKEN_SYMBOL=tok
LOWER_SYMBOL=lc
BPE_SYMBOL=bpe
TAG_SYMBOL=tagged
SYMBOLS=${TOKEN_SYMBOL} ${LOWER_SYMBOL} ${BPE_SYMBOL}

# parameters should be configured
# SCRIPTS_DIR is the path of "nmt_tools": https://github.com/YasirHuang/nmt_tools.git
SCRIPTS_DIR=~/work/tools
# ORIGINAL_DATA_PATH is the path of Multi30K data path, which contains "mmt_task1", "mmt_task2", "splits", "flickr30k_images", "Annotations" and "Sentences"
ORIGINAL_DATA_PATH=/home/xhuang/work/corpus/multi30k/original
# SOURCE_DIR is the path of Multi30K mmt_task1 original data directory, which contains "train/val/test2016.en/de"
SOURCE_DIR=${ORIGINAL_DATA_PATH}/mmt_task1/raw
# SPLITS_DIR
SPLITS_DIR=${ORIGINAL_DATA_PATH}/splits
# ANNOTATED_SENTENCE_PATH
ANNOTATED_DATA=${ORIGINAL_DATA_PATH}/annotations.zip
ANNOTATED_SENTENCE_PATH=${ORIGINAL_DATA_PATH}/Sentences
# image bounding box annotations dir (useless)
# BOUNDING_BOX_ANNOTATION_PATH=${ORIGINAL_DATA_PATH}/Annotations
# OUT_DIR is the path of the outputs of this script, default "./"
OUT_DIR=.
# languages to be processed, default: en, de. "fr" to be added.
LANGS=en de
JSON_OUT_SUFFIX=en.json
# other files
KEY_WORD_LIST_FILE=${OUT_DIR}/key_word_list.txt
KEY_WORD_POS_LIST_FILE=${OUT_DIR}/key_word_pos_list.txt

# original data file prefixes, include train/val/test
TRAIN_PREFIX=train.fixed
VAL_PREFIX=val
TEST_2016_PREFIX=test_2016_flickr
TEST_2017_PREFIX=test_2017_flickr
TEST_2018_PREFIX=test_2018_flickr
TEST_MSCOCO_PREFIX=test_2017_mscoco
TEST_PREFIX=${TEST_2016_PREFIX} ${TEST_2017_PREFIX} ${TEST_2018_PREFIX} ${TEST_MSCOCO_PREFIX}
VOCAB_PREFIX=vocab
# splits file names
TRAIN_SPLITS=train_images.txt
VAL_SPLITS=val_images.txt
TEST_SPLITS=test_images.txt
# iterable file prefixes
FILE_PREFIXS=$(TRAIN_PREFIX) ${VAL_PREFIX} ${TEST_PREFIX}
FILE_PREFIXS_TOKEN=$(foreach i, $(FILE_PREFIXS), $(i).$(TOKEN_SYMBOL))
FILE_PREFIXS_BPE=$(foreach i, $(FILE_PREFIXS_TOKEN), $(i).$(LOWER_SYMBOL))
# file prefixes for extracting entites for source sentences
TRAIN_JSON_OUT_PREFIX=${OUT_DIR}/${TRAIN_PREFIX}.${TOKEN_SYMBOL}.${LOWER_SYMBOL}
VAL_JSON_OUT_PREFIX=${OUT_DIR}/${VAL_PREFIX}.${TOKEN_SYMBOL}.${LOWER_SYMBOL}
TEST_JSON_OUT_PREFIX=${OUT_DIR}/${TEST_PREFIX}.${TOKEN_SYMBOL}.${LOWER_SYMBOL}

all: check tok lowercase bpe vocab add_entities
# check files
check:
ifneq ($(ORIGINAL_DATA_PATH), $(wildcard $(ORIGINAL_DATA_PATH)))
	echo "original data directory not found."
	exit 1
else
	echo "original data directory exists." 
endif
ifneq ($(SOURCE_DIR), $(wildcard $(SOURCE_DIR)))
	echo "source directory not found."
	exit 1
endif
ifneq ($(SPLITS_DIR), $(wildcard $(SPLITS_DIR)))
	echo "splits directory not found."
	exit 1
endif
ifneq ($(ANNOTATED_SENTENCE_PATH), $(wildcard $(ANNOTATED_SENTENCE_PATH)))
ifeq ($(ANNOTATED_DATA), $(wildcard $(ANNOTATED_DATA)))
	unzip $(ANNOTATED_DATA) -d $(ORIGINAL_DATA_PATH)
ifeq ($(ANNOTATED_SENTENCE_PATH), $(wildcard $(ANNOTATED_SENTENCE_PATH)))
	echo "annotated sentence files unzipped to original data directory."
endif
else
	echo "annotated zip file not found."
	exit 1
endif
endif
ifneq ($(KEY_WORD_LIST_FILE), $(wildcard $(KEY_WORD_LIST_FILE)))
	echo "key_word_list_file.txt not found."
	exit 1
endif
ifneq ($(KEY_WORD_POS_LIST_FILE), $(wildcard $(KEY_WORD_POS_LIST_FILE)))
	echo "key_word_pos_list_file.txt not found."
	exit 1
endif




tok:
	for l in $(LANGS); do \
		echo "do tokenize for language: $$l"; \
		for fp in $(FILE_PREFIXS); do \
			perl ${SCRIPTS_DIR}/tokenizer.perl -l $$l -no-escape < ${SOURCE_DIR}/$$fp.$$l > ${OUT_DIR}/$$fp.${TOKEN_SYMBOL}.$$l; \
		done; \
	done
lowercase:
	for l in $(LANGS); do \
		echo "do lowercase for language: $$l"; \
		for fp in $(FILE_PREFIXS_TOKEN); do \
			perl ${SCRIPTS_DIR}/lowercase.perl < ${OUT_DIR}/$$fp.$$l > ${OUT_DIR}/$$fp.$(LOWER_SYMBOL).$$l; \
		done; \
	done


BPE_OPS=10000
BPE_IN_FILE_PREFIX=${TRAIN_PREFIX}.${TOKEN_SYMBOL}.${LOWER_SYMBOL}
bpe: learn_bpe apply_bpe
learn_bpe:
	for l in $(LANGS); do \
		echo "learning bpe model for language: $$l"; \
		python ${SCRIPTS_DIR}/learn_bpe.py -s ${BPE_OPS} < ${OUT_DIR}/${BPE_IN_FILE_PREFIX}.$$l > ${OUT_DIR}/bpemodel.${BPE_IN_FILE_PREFIX}.$$l; \
	done
apply_bpe:
	for l in $(LANGS); do \
		echo "apply bpe model to language: $$l"; \
		for fp in $(FILE_PREFIXS_BPE); do \
			python ${SCRIPTS_DIR}/apply_bpe.py -c ${OUT_DIR}/bpemodel.${BPE_IN_FILE_PREFIX}.$$l < ${OUT_DIR}/$$fp.$$l > ${OUT_DIR}/$$fp.$(BPE_SYMBOL).$$l; \
		done; \
	done

vocab:
	for l in $(LANGS); do \
		echo "create vocabs for language: $$l"; \
		in_file_prefix=${OUT_DIR}/${TRAIN_PREFIX}; \
		out_file_prefix=${OUT_DIR}/${VOCAB_PREFIX}; \
		for s in ${SYMBOLS}; do \
			in_file_prefix=$$in_file_prefix.$$s; \
			out_file_prefix=$$out_file_prefix.$$s; \
			python ${SCRIPTS_DIR}/vocab.py --in_dir=${OUT_DIR}/ --out_dir=${OUT_DIR}/ --in_file_name=$$in_file_prefix --out_file_name=$$out_file_prefix --out_counted_file_name=$$out_file_prefix.counted --lang=$$l; \
		done; \
	done
	in_file_prefix=${OUT_DIR}/${VOCAB_PREFIX};\
	for s in ${SYMBOLS}; do \
		in_file_prefix=$$in_file_prefix.$$s; \
		python ${SCRIPTS_DIR}/merge_vocab.py --vocab_prefix=$$in_file_prefix -langs $(LANGS); \
	done

add_entities: extract_entities add_tag_to_json_data bpe_to_json

# inputs
EXTRACT_IN_TRAIN=${OUT_DIR}/${TRAIN_PREFIX}.${TOKEN_SYMBOL}.${LOWER_SYMBOL}.en
EXTRACT_IN_VAL=${OUT_DIR}/${VAL_PREFIX}.${TOKEN_SYMBOL}.${LOWER_SYMBOL}.en
EXTRACT_IN_TEST=${OUT_DIR}/${TEST_PREFIX}.${TOKEN_SYMBOL}.${LOWER_SYMBOL}.en
# outputs
EXTRACT_OUT_TRAIN=${TRAIN_JSON_OUT_PREFIX}.${JSON_OUT_SUFFIX}
EXTRACT_OUT_VAL=${VAL_JSON_OUT_PREFIX}.${JSON_OUT_SUFFIX}
EXTRACT_OUT_TEST=${TEST_JSON_OUT_PREFIX}.${JSON_OUT_SUFFIX}
extract_entities:
	python ${SCRIPTS_DIR}/extract_m30k_task1_noted_sentences.py \
		--split_file=${SPLITS_DIR}/${TRAIN_SPLITS} \
		--sentences_file=${EXTRACT_IN_TRAIN} \
		--annoted_sentence_path=${ANNOTATED_SENTENCE_PATH} \
		--output_file=${EXTRACT_OUT_TRAIN}
	python ${SCRIPTS_DIR}/extract_m30k_task1_noted_sentences.py \
		--split_file=${SPLITS_DIR}/${VAL_SPLITS} \
		--sentences_file=${EXTRACT_IN_VAL} \
		--annoted_sentence_path=${ANNOTATED_SENTENCE_PATH} \
		--output_file=${EXTRACT_OUT_VAL}
	python ${SCRIPTS_DIR}/extract_m30k_task1_noted_sentences.py \
		--split_file=${SPLITS_DIR}/${TEST_SPLITS} \
		--sentences_file=${EXTRACT_IN_TEST} \
		--annoted_sentence_path=${ANNOTATED_SENTENCE_PATH} \
		--output_file=${EXTRACT_OUT_TEST}
# outputs
ADD_OUT_TRAIN=${TRAIN_JSON_OUT_PREFIX}.${TAG_SYMBOL}.${JSON_OUT_SUFFIX}
ADD_OUT_VAL=${VAL_JSON_OUT_PREFIX}.${TAG_SYMBOL}.${JSON_OUT_SUFFIX}
ADD_OUT_TEST=${TEST_JSON_OUT_PREFIX}.${TAG_SYMBOL}.${JSON_OUT_SUFFIX}
add_tag_to_json_data:
	python ${SCRIPTS_DIR}/add_tag_to_json_data.py \
		--split_file=${SPLITS_DIR}/${TRAIN_SPLITS} \
		--json_file=${EXTRACT_OUT_TRAIN} \
		--out_json_file=${ADD_OUT_TRAIN} \
		--key_word_list_file=${KEY_WORD_LIST_FILE} \
		--key_word_pos_list_file=${KEY_WORD_POS_LIST_FILE}
	python ${SCRIPTS_DIR}/add_tag_to_json_data.py \
		--split_file=${SPLITS_DIR}/${VAL_SPLITS} \
		--json_file=${EXTRACT_OUT_VAL} \
		--out_json_file=${ADD_OUT_VAL} \
		--key_word_list_file=${KEY_WORD_LIST_FILE} \
		--key_word_pos_list_file=${KEY_WORD_POS_LIST_FILE}
	python ${SCRIPTS_DIR}/add_tag_to_json_data.py \
		--split_file=${SPLITS_DIR}/${TEST_SPLITS} \
		--json_file=${EXTRACT_OUT_TEST} \
		--out_json_file=${ADD_OUT_TEST} \
		--key_word_list_file=${KEY_WORD_LIST_FILE} \
		--key_word_pos_list_file=${KEY_WORD_POS_LIST_FILE}
# outputs
BPE_OUT_TRAIN=${TRAIN_JSON_OUT_PREFIX}.${BPE_SYMBOL}.${TAG_SYMBOL}.${JSON_OUT_SUFFIX}
BPE_OUT_VAL=${VAL_JSON_OUT_PREFIX}.${BPE_SYMBOL}.${TAG_SYMBOL}.${JSON_OUT_SUFFIX}
BPE_OUT_TEST=${TEST_JSON_OUT_PREFIX}.${BPE_SYMBOL}.${TAG_SYMBOL}.${JSON_OUT_SUFFIX}
bpe_to_json:
	python ${SCRIPTS_DIR}/apply_bpe_to_aligned_data.py \
		--bpe_sentence_file=${TRAIN_JSON_OUT_PREFIX}.${BPE_SYMBOL}.en \
		--split_file=${SPLITS_DIR}/${TRAIN_SPLITS} \
		--json_file=${ADD_OUT_TRAIN} \
		--out_json_file=${BPE_OUT_TRAIN}
	python ${SCRIPTS_DIR}/apply_bpe_to_aligned_data.py \
		--bpe_sentence_file=${VAL_JSON_OUT_PREFIX}.${BPE_SYMBOL}.en \
		--split_file=${SPLITS_DIR}/${VAL_SPLITS} \
		--json_file=${ADD_OUT_VAL} \
		--out_json_file=${BPE_OUT_VAL}
	python ${SCRIPTS_DIR}/apply_bpe_to_aligned_data.py \
		--bpe_sentence_file=${TEST_JSON_OUT_PREFIX}.${BPE_SYMBOL}.en \
		--split_file=${SPLITS_DIR}/${TEST_SPLITS} \
		--json_file=${ADD_OUT_TEST} \
		--out_json_file=${BPE_OUT_TEST}

clean:
	rm *$(BPE_SYMBOL)* *$(LOWER_SYMBOL)* *$(TOKEN_SYMBOL)*
